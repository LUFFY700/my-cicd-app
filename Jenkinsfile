pipeline {
    agent any

    environment {
        // IMPORTANT: Make sure this name matches the Node.js tool name in Jenkins (e.g., 'NodeJS_24.2.0')
        NODE_VERSION = 'NodeJS_24.2.0' // <--- Update this if your Node.js tool name is different!

        IMAGE_TAG = "v1.0"
        // It's generally safer to use built-in BRANCH_NAME for Multibranch Pipelines if possible,
        // but this 'sh' command should also work if git is available early enough.
        // BRANCH_NAME = sh(returnStdout: true, script: "git rev-parse --abbrev-ref HEAD").trim()
    }

    tools {
        // This makes 'npm' available in the PATH for subsequent steps
        nodejs env.NODE_VERSION
    }

    stages {
        // Checkout the source code from the SCM (Git repository)
        stage('Checkout') {
            steps {
                // checkout scm is good for Multibranch Pipelines
                checkout scm
            }
        }

        // Install application dependencies
        stage('Build') {
            steps {
                // 'npm' will now be found because of the 'tools' block
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
                    // Get the branch name directly from Jenkins environment variables
                    // Assuming BRANCH_NAME is set by Multibranch Pipeline or a previous step
                    def currentBranchName = env.BRANCH_NAME // Using env.BRANCH_NAME
                    if (!currentBranchName) {
                         // Fallback if env.BRANCH_NAME isn't set, might need to re-add sh 'git rev-parse'
                         // But for multibranch, env.BRANCH_NAME should be present.
                         currentBranchName = sh(returnStdout: true, script: "git rev-parse --abbrev-ref HEAD").trim()
                    }

                    // Use branch-specific image names
                    def imageName = (currentBranchName == 'main') ? "nodemain:${env.IMAGE_TAG}" : "nodedev:${env.IMAGE_TAG}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        // Deploy the application in a Docker container
        stage('Deploy') {
            steps {
                script {
                    // Get the branch name directly from Jenkins environment variables
                    def currentBranchName = env.BRANCH_NAME // Using env.BRANCH_NAME
                    if (!currentBranchName) {
                         currentBranchName = sh(returnStdout: true, script: "git rev-parse --abbrev-ref HEAD").trim()
                    }

                    // Use branch-specific image names and ports
                    def imageName = (currentBranchName == 'main') ? "nodemain:${env.IMAGE_TAG}" : "nodedev:${env.IMAGE_TAG}"
                    def port = (currentBranchName == 'main') ? "3000" : "3001" // Ports are now correctly string literals

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
