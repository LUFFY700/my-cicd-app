pipeline {
    agent any

    environment {
        NODE_VERSION = 'NodeJS_24.2.0'
        IMAGE_TAG = "v1.0" // Consider making this dynamic, e.g., ${BUILD_NUMBER}
        // BRANCH_NAME is automatically provided by Multibranch Pipeline projects
    }

    tools {
        nodejs env.NODE_VERSION
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // env.BRANCH_NAME is the standard way to get the branch name in Multibranch Pipelines
                    // No need for a fallback 'git rev-parse' here, it's already set.
                    def currentBranchName = env.BRANCH_NAME

                    // Define image name based on branch, could be more elaborate
                    def imageName
                    if (currentBranchName == 'main') {
                        imageName = "my-app-main:${env.IMAGE_TAG}"
                    } else if (currentBranchName == 'dev') {
                        imageName = "my-app-dev:${env.IMAGE_TAG}"
                    } else {
                        // For feature branches, use the branch name in the tag or image name
                        // Be careful with special characters in branch names for Docker tags.
                        // Often, you might slugify or hash the branch name.
                        // For simplicity here, let's just use a generic 'feature' prefix for now
                        // or even directly use the branch name in the tag.
                        imageName = "my-app-${currentBranchName.toLowerCase().replaceAll(/[^a-z0-9_.-]/, '-')}:${env.IMAGE_TAG}"
                    }

                    echo "Building Docker image: ${imageName}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def currentBranchName = env.BRANCH_NAME

                    def imageName // Image name must match what was built in the previous stage
                    def deployPort // Port to map on the host

                    if (currentBranchName == 'main') {
                        imageName = "my-app-main:${env.IMAGE_TAG}"
                        deployPort = "3000" // Main app on 3000
                    } else if (currentBranchName == 'dev') {
                        imageName = "my-app-dev:${env.IMAGE_TAG}"
                        deployPort = "3001" // Dev app on 3001
                    } else {
                        // For feature branches, you might deploy them to unique ports or environments
                        // For simplicity, let's say all other branches get port 3002
                        // Or, you could generate a unique port based on the branch hash for more complex setups.
                        imageName = "my-app-${currentBranchName.toLowerCase().replaceAll(/[^a-z0-9_.-]/, '-')}:${env.IMAGE_TAG}"
                        deployPort = "3002" // Feature branches on 3002
                    }

                    echo "Stopping any running containers for ${imageName}..."
                    sh """
                        docker ps -q --filter "ancestor=${imageName}" | xargs -r docker stop || true
                        docker ps -aq --filter "ancestor=${imageName}" | xargs -r docker rm || true
                    """

                    echo "Deploying new container for ${imageName} on host port ${deployPort}..."
                    sh "docker run -d -p ${deployPort}:3000 ${imageName}" // Assuming internal container port is 3000
                    
                    // Optional cleanup of unused images
                    sh 'docker image prune -f || true'
                }
            }
        }
    }

    post {
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
