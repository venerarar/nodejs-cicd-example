pipeline {
    agent any

    environment {
        DOCKER_USERNAME_CONST = 'venerara' // Just for reference, avoid collision
        IMAGE_NAME = "nodejs-app"
        IMAGE_FULL_TAG = "${DOCKER_USERNAME_CONST}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
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
                bat "docker build -t ${IMAGE_FULL_TAG} ."
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    bat '''
                        echo %DOCKER_HUB_PASSWORD% | docker login -u %DOCKER_HUB_USERNAME% --password-stdin
                        docker push ${IMAGE_FULL_TAG}
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy Docker Container') {
            steps {
                bat '''
                    docker stop ${CONTAINER_NAME} || exit 0
                    docker rm ${CONTAINER_NAME} || exit 0
                    docker run -d -p 3000:3000 --name ${CONTAINER_NAME} ${IMAGE_FULL_TAG}
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline finished successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
