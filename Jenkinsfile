// Use a variable to define the image name once, which makes it easier to manage.
def DOCKER_REPO = 'barkhalahori/dockerpipeline'

pipeline {
    agent any

    stages {
        stage("Checkout") {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/barkhalahori/dockerpipeline']])
                echo 'Code checked out successfully.'
            }
        }

        stage("Build Image") {
            // Define the dynamic tag (e.g., using the build number) for versioning
            environment {
                // Defines a Groovy variable to hold the unique tag
		DOCKER_HOST = 'tcp://host.docker.internal:2375'
                LATEST_TAG = "${DOCKER_REPO}:latest"
            }
            steps {
                script {
                    if (isUnix()) {
                        sh 'docker build -t barkhalahori/dockerpipeline .' 
                    } else {
                        bat 'docker build -t barkhalahori/dockerpipeline ." 
                    }
                }
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub', 
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    
                    script {
                        def login_cmd = "docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}"

                        // 2. Push the uniquely tagged image
                        def push_tagged_cmd = 'docker push barkhalahori/dockerpipeline'

                        

                        // --- Execution ---
                        if (isUnix()) {
                            sh login_cmd
                            sh push_tagged_cmd
                            sh 'docker logout' // Logout does not require variables
                        } else {
                            bat login_cmd
                            bat push_tagged_cmd
                            bat 'docker logout' // Logout does not require variables
                        }
                    }
                }
            }
        }
    }
}