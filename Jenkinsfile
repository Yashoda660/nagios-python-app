pipeline {
    agent any

    environment {
        DOCKER_USER    = "yash09876"
        IMAGE_REPO     = "nagios-python-app"
        CONTAINER_NAME = "nagios-python-container"
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t ${DOCKER_USER}/${IMAGE_REPO}:${IMAGE_TAG} .'
            }
        }

        stage('Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'U',
                    passwordVariable: 'P'
                )]) {
                    sh 'echo $P | docker login -u $U --password-stdin'
                }
            }
        }

        stage('Push') {
            steps {
                sh 'docker push ${DOCKER_USER}/${IMAGE_REPO}:${IMAGE_TAG}'
            }
        }

        stage('Run') {
            steps {
                sh '''
                docker rm -f nagios-python-container || true
                docker run -d --name nagios-python-container \
                -p 5001:5000 ${DOCKER_USER}/${IMAGE_REPO}:${IMAGE_TAG}
                '''
            }
        }
    }
}
