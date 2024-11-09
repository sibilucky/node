pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "my-app-image"
        DOCKER_TAG = "latest"
        REGISTRY_URL = "sibisam2301"  // Docker registry URL (optional)
        SERVER_IP = "your-server-ip"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout([$class: 'GitSCM', 
                          branches: [[name: '*/main']], 
                          userRemoteConfigs: [[url: 'https://github.com/sibilucky/node.git', credentialsId: 'my-git-credentials']]])
            }
        }

       
        stage('Build Docker Image') {
            steps {
                script {
                    // Ensure docker is available
                    sh 'docker --version'
                    // Your Docker build command
                    sh 'docker build -t my-app-image:latest .'
                }
            }
        }
    


        stage('Push Docker Image to Docker Hub') {
            steps {
                // Push the Docker image to Docker Hub (if applicable)
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh """
                    docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
                    docker push my-app-image:latest
                    """
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                // SSH into the server and deploy the Docker container
                sh """
                ssh user@${SERVER_IP} 'docker pull  my-app-image:latest&& \
                docker stop my-app-image container || true && \
                docker rm my-app-image container || true && \
                docker run -d -p 80:3000 --name my-app-image container my-app-image:latest'
                """
            }
        }
    }

    post {
        always {
            // Clean up Docker images and containers if necessary
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
