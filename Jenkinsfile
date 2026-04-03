pipeline {
    agent any

    environment {
        IMAGE_NAME = "yash09876/nagios-python-app"
        REPO       = "yash09876/nagios-python-app"
    }

    stages {

        stage('Build & Push Image') {
            steps {
                script {
                    docker.withRegistry(
                        'https://registry.hub.docker.com',
                        'dockerhub-creds'
                    ) {
                        sh """
                        docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                        docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                        """

                        docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push()
                        docker.image("${IMAGE_NAME}:latest").push()
                    }
                }
            }
        }

        stage('Delete Old Tags - Keep Last 3') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                    echo "Deleting old Docker Hub tags (keeping last 3)"

                    TAGS=$(curl -s -u $DOCKER_USER:$DOCKER_TOKEN \
                      "https://hub.docker.com/v2/repositories/${REPO}/tags?page_size=100" \
                      | jq -r '.results[].name' | tail -n +4)

                    for TAG in $TAGS; do
                        echo "Deleting tag: $TAG"
                        curl -s -X DELETE -u $DOCKER_USER:$DOCKER_TOKEN \
                          "https://hub.docker.com/v2/repositories/${REPO}/tags/$TAG/"
                    done
                    '''
                }
            }
        }

    }   // <-- closes stages
}       // <-- closes pipeline
