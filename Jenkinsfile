pipeline {
    agent any

    environment {
        IMAGE_TAG = "v1.0"
        BRANCH_NAME = sh(returnStdout: true, script: "git rev-parse --abbrev-ref HEAD").trim()
    }

    stages {
        // Checkout the source code from the SCM (Git repository)
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // Install application dependencies
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        // Run tests to verify the application
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        // Build the Docker image dynamically based on the branch
        stage('Build Docker Image') {
            steps {
                script {
                    // Use branch-specific image names
                    def imageName = (BRANCH_NAME == 'main') ? "nodemain:${IMAGE_TAG}" : "nodedev:${IMAGE_TAG}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        // Deploy the application in a Docker container
        stage('Deploy') {
            steps {
                script {
                    // Use branch-specific image names and ports
                    def imageName = (BRANCH_NAME == 'main') ? "nodemain:${IMAGE_TAG}" : "nodedev:${IMAGE_TAG}"
                    def port = (BRANCH_NAME == 'main') ? "3000" : "3001"

                    // Stop and remove any existing container for this branch
                    sh """
                        echo "Stopping any running containers for ${imageName}..."
                        docker ps -q --filter "ancestor=${imageName}" | xargs -r docker stop || true
                        docker ps -aq --filter "ancestor=${imageName}" | xargs -r docker rm || true

                        echo "Deploying new container for ${imageName}..."
                        docker run -d -p ${port}:3000 ${imageName}
                    """

                    // Optional cleanup of unused images to free space
                    sh 'docker image prune -f || true'
                }
            }
        }
    }

    post {
        // Notify or perform cleanup after successful or failed pipelines
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Deployment was successful!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
