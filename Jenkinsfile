// Use the correct Docker Hub repository name (USER/REPO)
def DOCKER_REPO = 'barkha2911/dockerpipeline'
def IMAGE_TAG = "${DOCKER_REPO}:${env.BUILD_NUMBER}" // Unique tag for this build (e.g., barkha2911/dockerpipeline:15)
def LATEST_TAG = "${DOCKER_REPO}:latest"             // The 'latest' tag

pipeline {
    agent any
    
    // Environment variables that apply to the whole pipeline (or just where needed)
    environment {
        // You only need DOCKER_HOST if your Jenkins agent is running as a container
        // and you're using 'Docker-in-Docker' or 'Docker-outside-of-Docker'.
        // DOCKER_HOST = 'tcp://host.docker.internal:2375' 
    }

    stages {
        stage("Checkout") {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/barkhalahori/dockerpipeline']])
                echo 'Code checked out successfully.'
            }
        }

        stage("Build & Tag Image") {
            steps {
                script {
                    // 1. Build and tag with the unique BUILD_NUMBER
                    // 2. Also tag it as 'latest'
                    def build_cmd = "docker build -t ${IMAGE_TAG} -t ${LATEST_TAG} ."
                    
                    if (isUnix()) {
                        sh build_cmd
                    } else {
                        bat build_cmd
                    }
                }
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub', // Ensure this ID is correct in Jenkins
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        
                    script {
                        // 1. Login
                        def login_cmd = "docker login -u ${DOCKERHUB_USERNAME} -p ${DOCKERHUB_PASSWORD}"

                        // 2. Define the push commands
                        def push_versioned_cmd = "docker push ${IMAGE_TAG}" // Push the specific version
                        def push_latest_cmd    = "docker push ${LATEST_TAG}"  // Push the latest tag

                        // --- Execution ---
                        if (isUnix()) {
                            sh login_cmd
                            sh push_versioned_cmd // Push the versioned tag
                            sh push_latest_cmd     // Push the latest tag
                            sh 'docker logout'
                        } else {
                            bat login_cmd
                            bat push_versioned_cmd
                            bat push_latest_cmd
                            bat 'docker logout'
                        }
                    }
                }
            }
        }
        
        stage('Run/Test Container') {
            steps {
                echo "Testing image ${IMAGE_TAG}"
                // Add your testing step here, for example:
                sh "docker run -d -p 8080:3000 --name ci-test-app ${IMAGE_TAG}"
                // Add a sleep or a curl command to hit the container endpoint
                // sh "sleep 5"
                // sh "curl -s http://localhost:8080 | grep 'Welcome to Express'" 
                sh "docker stop ci-test-app"
                sh "docker rm ci-test-app"
            }
        }
    }
}