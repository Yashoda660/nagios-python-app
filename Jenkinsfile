pipeline {
    agent any

    environment {
        IMAGE_NAME = "yash09876/nagios-python-app"
        CONTAINER_NAME = "nagios-python-container"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo 'Code already checked out from GitHub'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Docker Login') {
            steps {
                sh "docker login -u yash09876 -p YOUR_DOCKER_HUB_TOKEN"
            }
        }

        stage('Push Image') {
            steps {
                sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
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
}
