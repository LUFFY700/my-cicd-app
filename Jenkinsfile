pipeline {
    agent any // This means any available Jenkins agent can run this pipeline

    environment {
        // This name must match the NodeJS tool name you configured in Jenkins (e.g., NodeJS_24.2.0)
        // IMPORTANT: Update 'NodeJS_24.2.0' below if your Jenkins NodeJS tool has a different name!
        NODE_VERSION = 'NodeJS_24.2.0' 

        // Determine Docker image name and application port based on the branch
        DOCKER_IMAGE_NAME = "node-app-${BRANCH_NAME == 'main' ? 'main' : 'dev'}"
        DOCKER_IMAGE_TAG = 'v1.0' // Our default image tag
        APP_PORT = (BRANCH_NAME == 'main') ? "3000" : "3001" // Main on 3000, Dev on 3001
        CONTAINER_NAME = "app-${BRANCH_NAME}" // Unique container name per branch
    }

    tools {
        // Use the NodeJS tool configured globally in Jenkins, referenced by its name
        nodejs env.NODE_VERSION
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    echo "Checking out ${env.BRANCH_NAME} branch from Git..."
                    // Git clone operation.
                    // IMPORTANT: REPLACE 'https://github.com/LUFFY700/my-cicd-app' with YOUR ACTUAL GitHub repository URL.
                    // IMPORTANT: REPLACE 'github-pipeline-pat' with YOUR EXACT Credentials ID you set in Jenkins!
                    git branch: "${env.BRANCH_NAME}", credentialsId: 'github-pipeline-pat', url: 'https://github.com/LUFFY700/my-cicd-app' 
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    echo "Installing Node.js dependencies for ${env.BRANCH_NAME} branch..."
                    sh 'npm install'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    echo "Running tests for ${env.BRANCH_NAME} branch..."
                    sh 'npm test'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG} for ${env.BRANCH_NAME} branch..."
                    // Ensure the Dockerfile is in the current context (root of the workspace)
                    dir("${WORKSPACE}") {
                        docker.build "${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}", '.' // '.' indicates Dockerfile is in the current directory
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    echo "Deploying application for ${env.BRANCH_NAME} branch on port ${env.APP_PORT}..."

                    // Stop and remove any existing container for this branch to ensure a clean deploy
                    def existingContainer = sh(returnStdout: true, script: "docker ps -a --filter 'name=${env.CONTAINER_NAME}' --format '{{.ID}}'").trim()
                    if (existingContainer) {
                        echo "Stopping and removing existing container: ${env.CONTAINER_NAME}"
                        sh "docker stop ${env.CONTAINER_NAME} || true" // '|| true' prevents script from failing if container is not found/running
                        sh "docker rm ${env.CONTAINER_NAME} || true"
                    } else {
                        echo "No existing container ${env.CONTAINER_NAME} found to stop/remove."
                    }

                    // Run the new Docker container
                    echo "Running new container: ${env.CONTAINER_NAME} on port ${env.APP_PORT}"
                    sh "docker run -d --name ${env.CONTAINER_NAME} -p ${env.APP_PORT}:3000 ${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}"
                    echo "Application deployed on http://localhost:${env.APP_PORT}"
                }
            }
        }
    } // End of stages
}
