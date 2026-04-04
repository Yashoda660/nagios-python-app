pipeline {
    agent any

    environment {
        DOCKER_USER    = "yash09876"
        IMAGE_REPO     = "nagios-python-app"
        CONTAINER_NAME = "nagios-python-container"
        KEEP_IMAGES    = 3
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    stages {

        stage('Show Build Info') {
            steps {
                echo "Jenkins Build Number = ${BUILD_NUMBER}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
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

        stage('Push Docker Image') {
            steps {
                sh """
                docker push ${DOCKER_USER}/${IMAGE_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy Latest Image') {
            steps {
                sh """
                docker rm -f ${CONTAINER_NAME} || true
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
                    JWT_TOKEN=\$(curl -s -X POST https://hub.docker.com/v2/users/login/ \
                      -H "Content-Type: application/json" \
                      -d '{ "username": "${DOCKER_USER}", "password": "${DOCKER_PAT}" }' | jq -r .token)

                    TAGS=\$(curl -s -H "Authorization: Bearer \$JWT_TOKEN" \
                      https://hub.docker.com/v2/repositories/${DOCKER_USER}/${IMAGE_REPO}/tags/?page_size=100 \
                      | jq -r '.results | sort_by(.last_updated) | reverse | .[].name')

                    COUNT=0
                    for TAG in \$TAGS; do
                        COUNT=\$((COUNT+1))
                        if [ \$COUNT -gt ${KEEP_IMAGES} ]; then
                            curl -s -X DELETE \
                              -H "Authorization: Bearer \$JWT_TOKEN" \
                              https://hub.docker.com/v2/repositories/${DOCKER_USER}/${IMAGE_REPO}/tags/\$TAG/
                        fi
                    done
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ SUCCESS: Only last 3 Docker Hub tags retained"
        }
        failure {
            echo "❌ FAILURE: Pipeline failed"
        }
    }
}

