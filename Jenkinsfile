pipeline {
    agent any // Or a specific agent label if you have one, e.g., agent { label 'docker-host' }

    environment {
        // Replace with your Docker Hub username and repository name
        DOCKER_HUB_USERNAME = 'venerara'
        IMAGE_NAME = "nodejs-app"
        IMAGE_FULL_TAG = "${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
        CONTAINER_NAME = "nodejs-app-container"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/venerarar/nodejs-cicd-example.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_FULL_TAG} ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                        sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                        sh "docker push ${IMAGE_FULL_TAG}"
                        sh "docker logout" // Good practice to logout
                    }
                }
            }
        }

        stage('Deploy Docker Container') {
            steps {
                script {
                    // Stop and remove existing container if it's running
                    echo "Stopping and removing existing container (if any)..."
                    sh "docker stop ${CONTAINER_NAME} || true"
                    sh "docker rm ${CONTAINER_NAME} || true"

                    // Run the new container
                    echo "Running new container..."
                    sh "docker run -d -p 3000:3000 --name ${CONTAINER_NAME} ${IMAGE_FULL_TAG}"
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Clean up workspace
        }
        success {
            echo 'Pipeline finished successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}