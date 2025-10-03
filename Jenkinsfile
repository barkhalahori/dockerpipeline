// Use a variable to define the image name once, which makes it easier to manage.
def DOCKER_REPO = 'barkhalahori/dockerpipeline'

pipeline {
    agent any

    stages {
        stage("Checkout") {
            steps {
                // Ensure the 'github' credential ID is correct
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/barkhalahori/dockerpipeline.git']])
                echo 'Code checked out successfully.'
            }
        }

        stage("Build Image") {
            // Define the dynamic tag (e.g., using the build number) for versioning
            environment {
                // Defines a Groovy variable to hold the unique tag
                IMAGE_TAG = "${DOCKER_REPO}:${env.BUILD_NUMBER}" 
                LATEST_TAG = "${DOCKER_REPO}:latest"
            }
            steps {
                script {
                    if (isUnix()) {
                        // Use double quotes (") for variable interpolation
                        sh "docker build -t ${IMAGE_TAG} ." 
                    } else {
                        // Use double quotes (") for variable interpolation in bat/Windows
                        bat "docker build -t ${IMAGE_TAG} ." 
                    }
                }
            }
        }

        stage("Docker Push") {
            steps {
                // WARNING: There was a missing quote on credentialsId: 'dockerhub'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub', // Assuming 'dockerhub' is the correct ID
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    
                    script {
                        // 1. Login
                        // Use double quotes (") for the variables
                        def login_cmd = "docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}"

                        // 2. Push the uniquely tagged image
                        def push_tagged_cmd = "docker push ${IMAGE_TAG}"

                        // 3. Tag the build image as 'latest' for convenience
                        def tag_latest_cmd = "docker tag ${IMAGE_TAG} ${LATEST_TAG}"

                        // 4. Push the 'latest' tag
                        def push_latest_cmd = "docker push ${LATEST_TAG}"

                        // --- Execution ---
                        if (isUnix()) {
                            sh login_cmd
                            sh push_tagged_cmd
                            sh tag_latest_cmd
                            sh push_latest_cmd 
                            sh 'docker logout' // Logout does not require variables
                        } else {
                            bat login_cmd
                            bat push_tagged_cmd
                            bat tag_latest_cmd
                            bat push_latest_cmd
                            bat 'docker logout' // Logout does not require variables
                        }
                    }
                }
            }
        }
    }
}