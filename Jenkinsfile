pipeline {
    agent any

    environment {
        IMAGE_NAME     = "yash09876/nagios-python-app"
        CONTAINER_NAME = "nagios-python-container"
    }

    stages {

        // ✅ Checkout happens automatically from SCM
        stage('SCM Checkout (Auto)') {
            steps {
                echo 'Source code already checked out by Jenkins'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry(
                        'https://registry.hub.docker.com',
                        'dockerhub-creds'
                    ) {
                        docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push()
                    }
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker stop ${CONTAINER_NAME} || true
                docker rm ${CONTAINER_NAME} || true

                docker run -d --restart unless-stopped \
                  --name ${CONTAINER_NAME} \
                  ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Build, Push and Deployment Successful"
        }
        failure {
            echo "❌ Pipeline Failed"
        }
    }
}
