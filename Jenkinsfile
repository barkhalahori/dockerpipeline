// Define a variable for the current application version
def APP_VERSION = "1.0.${env.BUILD_NUMBER}"

pipeline {
    agent any // This runs the pipeline on any available Jenkins agent

    stages {
        
        // =================================================================
        stage('Initialize & Setup') {
            steps {
                echo "Starting Pipeline for Application Version: ${APP_VERSION}"
                
                // Use the 'script' block for more complex Groovy logic
                script {
                    // Check if the current environment is Unix-like (most agents)
                    if (isUnix()) {
                        sh "echo '--- Starting build on a Unix Agent ---'"
                    } else {
                        // Handle Windows environments if needed (using 'bat' command)
                        bat "echo --- Starting build on a Windows Agent ---"
                    }
                }
            }
        }
        
        // =================================================================
        stage('Create Artifacts') {
            steps {
                echo "Entering Create Artifacts stage..."
                
                // Create a temporary file (a mock "artifact") in the workspace
                sh "echo 'Build successful for version ${APP_VERSION}' > build_info.txt"
                sh "echo 'File created at: ${WORKSPACE}'"
            }
        }
        
        // =================================================================
        stage('Validate Artifact') {
            steps {
                echo "Entering Validate Artifact stage..."
                
                // Read the contents of the file created in the previous stage
                script {
                    // Load the content of the file into a Groovy variable
                    def artifact_content = readFile('build_info.txt').trim()
                    
                    echo "File Content: ${artifact_content}"
                    
                    // Simple validation check
                    if (artifact_content.contains(APP_VERSION)) {
                        echo "Validation SUCCESS: Artifact content contains the correct version."
                    } else {
                        // fail the build if validation is critical
                        error "Validation FAILURE: Artifact content is missing the version!"
                    }
                }
            }
        }

        // =================================================================
        stage('Deploy Notification') {
            steps {
                echo "Entering Deploy Notification stage..."
                echo "The build process is complete. The application is ready for deployment."
            }
        }
    }
    
    // =================================================================
    post {
        // Runs after every build attempt, regardless of the result
        always {
            echo 'Cleanup: The pipeline has finished executing.'
            // Optional: Clean up workspace files
            // cleanWs()
        }
        // Only runs if the build was successful (Status: SUCCESS)
        success {
            echo "SUCCESS! Pipeline completed without errors. Version ${APP_VERSION} is ready."
        }
        // Only runs if the build failed (Status: FAILURE)
        failure {
            echo "FAILURE! Review the logs for errors."
        }
    }
}