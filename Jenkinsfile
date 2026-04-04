pipeline {
    agent any

    environment {
        DOCKER_USER = "jibishwaran07"
        IMAGE_NAME  = "python-ci-cd-app"
        REPO        = "${DOCKER_USER}/${IMAGE_NAME}"
        KEEP_TAGS   = 3
    }

    stages {

        stage('Build Image') {
            steps {
                script {
                    def IMAGE_TAG = "v${env.BUILD_NUMBER}"
                    env.IMAGE_TAG = IMAGE_TAG
                    sh "docker build -t ${REPO}:${IMAGE_TAG} ."
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
                        set -e
                        TOKEN=$(curl -s -X POST https://hub.docker.com/v2/users/login/ \
                          -H "Content-Type: application/json" \
                          -d "{\"username\":\"$DH_USER\",\"password\":\"$DH_PASS\"}" \
                          | jq -r .token)

                        TAGS=$(curl -s -H "Authorization: JWT $TOKEN" \
                          https://hub.docker.com/v2/repositories/'"$REPO"'/tags?page_size=100 \
                          | jq -r '.results | sort_by(.last_updated) | reverse | .[].name')

                        count=0
                        for tag in $TAGS; do
                          count=$((count+1))
                          if [ $count -gt 3 ]; then
                            echo "Deleting old tag: $tag"
                            curl -s -X DELETE \
                              -H "Authorization: JWT $TOKEN" \
                              https://hub.docker.com/v2/repositories/'"$REPO"'/tags/$tag/
                          fi
                        done
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Only latest 3 Docker Hub tags retained"
        }
    }
}
