pipeline {
    agent any

    environment {
        IMAGE_NAME     = "yash09876/nagios-python-app"
        CONTAINER_NAME = "nagios-python-container"
    }

    stages {

        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        // ✅ THIS IS WHERE YOUR CODE GOES
        stage('Push & Deploy') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-creds') {

                        docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push()

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
    }
}
