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
    }



        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t my-app-image:latest ."
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
                        docker tag my-app-image:latest sibisam2301/my-app-image:latest
                       docker push sibisam2301/my-app-image:latest                   
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
                        docker pull sibisam2301/my-app-image:latest && \
                        docker stop my-app-image-container || true && \
                        docker rm my-app-mage-container || true && \
                        docker run -d -p 80:3000 --name my-app-image-container sibisam2301/my-app-mage:latest
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
