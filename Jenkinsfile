pipeline {
    agent any

    environment {
        DOCKER_USER    = "yash09876"
        IMAGE_REPO     = "nagios-python-app"
        CONTAINER_NAME = "nagios-python-container"
 HEAD
        KEEP_IMAGES    = 3

        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    stages {
 HEAD
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


        stage('Show Build Info') {
            steps {
                echo "Jenkins Build Number = ${BUILD_NUMBER}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                echo "Building image ${DOCKER_USER}/${IMAGE_REPO}:${IMAGE_TAG}"
                docker build -t ${DOCKER_USER}/${IMAGE_REPO}:${IMAGE_TAG} .
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_LOGIN_USER',
                        passwordVariable: 'DOCKER_LOGIN_PASS'
                    )
                ]) {
                    sh """
                    echo "\$DOCKER_LOGIN_PASS" | docker login -u "\$DOCKER_LOGIN_USER" --password-stdin
                    """
                }
            }
        }

       HEAD
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
        stage('Push Docker Image') {
            steps {
                sh """
                echo "Pushing image ${DOCKER_USER}/${IMAGE_REPO}:${IMAGE_TAG}"
                docker push ${DOCKER_USER}/${IMAGE_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy Latest Image') {
            steps {
                sh """
                docker stop ${CONTAINER_NAME} || true
                docker rm ${CONTAINER_NAME} || true

                docker run -d \
                  --name ${CONTAINER_NAME} \
                  -p 5001:5000 \
                  ${DOCKER_USER}/${IMAGE_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Cleanup Old Docker Hub Tags (Keep Last 3 Only)') {
            steps {
                withCredentials([
                    string(credentialsId: 'dockerhub-token', variable: 'DOCKER_PAT')
                ]) {
                    sh """
                    echo "Authenticating to Docker Hub..."

                    JWT_TOKEN=\$(curl -s -X POST https://hub.docker.com/v2/users/login/ \
                      -H "Content-Type: application/json" \
                      -d '{ "username": "${DOCKER_USER}", "password": "${DOCKER_PAT}" }' \
                      | jq -r .token)

                    echo "Fetching tags..."

                    TAGS=\$(curl -s -H "Authorization: Bearer \$JWT_TOKEN" \
                      https://hub.docker.com/v2/repositories/${DOCKER_USER}/${IMAGE_REPO}/tags/?page_size=100 \
                      | jq -r '.results | sort_by(.last_updated) | reverse | .[].name')

                    COUNT=0
                    for TAG in \$TAGS; do
                        COUNT=\$((COUNT+1))
                        if [ \$COUNT -gt ${KEEP_IMAGES} ]; then
                            echo "Deleting old tag: \$TAG"
                            curl -s -X DELETE \
                              -H "Authorization: Bearer \$JWT_TOKEN" \
                              https://hub.docker.com/v2/repositories/${DOCKER_USER}/${IMAGE_REPO}/tags/\$TAG/
                        else
                            echo "Keeping tag: \$TAG"
                        fi
                    done
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ SUCCESS: Only latest 3 Docker Hub tags retained"
        }
        failure {
            echo "❌ FAILURE: Pipeline failed"
        }
    }
}
