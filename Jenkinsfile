pipeline {
    // Defines where the pipeline runs (on any available agent/node)
    agent any

    // Defines variables used throughout the script
    environment {
        // !!! Ensure 'github' is the ID of your GitHub credentials in Jenkins !!!
        GITHUB_CREDENTIAL_ID = 'github' 

        // !!! Ensure 'dockerhub' is the ID of your Docker Hub credentials in Jenkins !!!
        DOCKERHUB_CREDENTIAL_ID = 'dockerhub' 

        // !!! CHANGE THIS to your Docker Hub username/image name !!!
        DOCKER_IMAGE_NAME = 'barkha2911/jenkins'
    }

    stages {
        stage("Checkout") {
            steps {
                // Checkout the code using the GitHub credentials and URL defined in the environment or job config.
                // NOTE: Using checkout scm requires the 'Pipeline script from SCM' job definition.
                // The provided checkout step is complex and redundant if using 'Pipeline script from SCM'.
                // Using 'checkout scm' is cleaner for SCM-based pipelines:
                checkout scm
                echo 'Code checked out successfully.'
            }
        }

        stage("Build Image") {
            steps {
                script {
                    // Build the Docker image, tagging it with the Jenkins BUILD_NUMBER for versioning
                    def imageTag = "${env.DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                    
                    if (isUnix()) {
                        sh "docker build -t ${imageTag} ."
                    } else {
                        bat "docker build -t ${imageTag} ."
                    }
                }
            }
        }

        stage("Docker Push") {
            steps {
                // Use the withCredentials block to securely expose Docker Hub username/password
                withCredentials([usernamePassword(
                    credentialsId: env.DOCKERHUB_CREDENTIAL_ID,
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    
                    script {
                        def imageTag = "${env.DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                        def latestTag = "${env.DOCKER_IMAGE_NAME}:latest"
                        
                        if (isUnix()) {
                            // Login, push both the build number tag and 'latest' tag, then logout
                            sh "docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}"
                            sh "docker push ${imageTag}"
                            sh "docker tag ${imageTag} ${latestTag}"
                            sh "docker push ${latestTag}"
                            sh "docker logout"
                        } else {
                            // Windows/Bat commands
                            bat "docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}"
                            bat "docker push ${imageTag}"
                            bat "docker tag ${imageTag} ${latestTag}"
                            bat "docker push ${latestTag}"
                            bat "docker logout"
                        }
                    }
                }
            }
        }
    }
}