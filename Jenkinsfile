pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "my-app-image"
        DOCKER_TAG = "latest"
        REGISTRY_URL = "dockerhub-username"  // Docker registry URL (optional)
        SERVER_IP = "your-server-ip"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/your-repo/my-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            when {
                branch 'main'
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh """
                        docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${REGISTRY_URL}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push ${REGISTRY_URL}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        """
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    sh """
                    ssh user@${SERVER_IP} '
                        docker pull ${REGISTRY_URL}/${DOCKER_IMAGE}:${DOCKER_TAG} && \
                        docker stop my-app-container || true && \
                        docker rm my-app-container || true && \
                        docker run -d -p 80:3000 --name my-app-container ${REGISTRY_URL}/${DOCKER_IMAGE}:${DOCKER_TAG}
                    '
                    """
                }
            }
        }
    }

    post {
        always {
            sh 'docker system prune -f'
        }

        success {
            echo "Deployment successful!"
        }

        failure {
            echo "Deployment failed!"
        }
    }
}
