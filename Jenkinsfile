pipeline {
    agent any

    environment {
        IMAGE_NAME = "yash09876/nagios-python-app"
    }

    stages {

        stage('Build Image') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry(
                        'https://registry.hub.docker.com',
                        'dockerhub-creds'
                    ) {
                        docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push()
                        docker.image("${IMAGE_NAME}:latest").push()
                    }
                }
            }
        }

        stage('Show Last 3 Tags') {
            steps {
                sh '''
                echo "===== LAST 3 DOCKER TAGS ====="
                curl -s "https://hub.docker.com/v2/repositories/yash09876/nagios-python-app/tags?page_size=100" \
                | jq -r '.results | .[0:3] | .[].name'
                '''
            }
        }
    }
}
