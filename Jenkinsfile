pipeline {
    agent any

    environment {
        DOCKER_USER = "jibishwaran07"
        IMAGE_NAME  = "python-ci-cd-app"
        REPO        = "${DOCKER_USER}/${IMAGE_NAME}"
        KEEP_TAGS   = 3
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                script {
                    IMAGE_TAG = "v${env.BUILD_NUMBER}"
                    sh """
                      echo "Building image ${REPO}:${IMAGE_TAG}"
                      docker build -t ${REPO}:${IMAGE_TAG} .
                    """
                }
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    sh """
                      echo "\$DH_PASS" | docker login -u "\$DH_USER" --password-stdin
                      docker push ${REPO}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Cleanup Docker Hub Tags (Keep Last 3)') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    sh '''
                      echo "Getting Docker Hub JWT token"
                      TOKEN=$(curl -s -X POST https://hub.docker.com/v2/users/login/ \
                        -H "Content-Type: application/json" \
                        -d '{"username":"'"$DH_USER"'","password":"'"$DH_PASS"'"}' | jq -r .token)

                      echo "Fetching tags"
                      TAGS=$(curl -s -H "Authorization: JWT $TOKEN" \
                        https://hub.docker.com/v2/repositories/'$REPO'/tags?page_size=100 \
                        | jq -r '.results | sort_by(.last_updated) | reverse | .[].name')

                      i=0
                      for tag in $TAGS; do
                        i=$((i+1))
                        if [ $i -gt $KEEP_TAGS ]; then
                          echo "Deleting old tag: $tag"
                          curl -s -X DELETE \
                            -H "Authorization: JWT $TOKEN" \
                            https://hub.docker.com/v2/repositories/'$REPO'/tags/$tag/
                        fi
                      done
                    '''
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
