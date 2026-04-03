pipeline {
    agent any

    environment {
        IMAGE_NAME = "jibishwaran07/python-ci-cd-app"
        IMAGE_TAG  = "v${BUILD_NUMBER}"
        KEEP_TAGS  = "3"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/jibishwaran07/python-ci-cd-app.git'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      set -e
                      docker login -u "$DOCKER_USER" -p "$DOCKER_PASS"

                      echo "Building image $IMAGE_NAME:$IMAGE_TAG"
                      docker build -t $IMAGE_NAME:$IMAGE_TAG .
                      docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Cleanup Docker Hub Tags (Keep Last 3)') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKER_TOKEN')]) {
                    sh '''
                      set -e
                      USER="jibishwaran07"
                      REPO="python-ci-cd-app"

                      echo "Fetching tags from Docker Hub…"

                      TAGS=$(curl -s \
                        -H "Authorization: Bearer $DOCKER_TOKEN" \
                        "https://hub.docker.com/v2/repositories/$USER/$REPO/tags/?page_size=100" \
                        | python3 - <<'EOF'
import json,sys
data=json.load(sys.stdin)
tags=sorted(data["results"], key=lambda x: x["last_updated"], reverse=True)
for t in tags:
    print(t["name"])
EOF
)

                      COUNT=0
                      for TAG in $TAGS; do
                        COUNT=$((COUNT+1))
                        if [ "$COUNT" -gt "$KEEP_TAGS" ]; then
                          echo "Deleting old tag: $TAG"
                          curl -s -X DELETE \
                            -H "Authorization: Bearer $DOCKER_TOKEN" \
                            "https://hub.docker.com/v2/repositories/$USER/$REPO/tags/$TAG/"
                        fi
                      done
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                      kubectl set image deployment/python-ci-cd-app \
                      flask-app=$IMAGE_NAME:$IMAGE_TAG
                      kubectl rollout status deployment/python-ci-cd-app
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo "Deployment failed – rolling back"
            sh 'kubectl rollout undo deployment/python-ci-cd-app'
        }
    }
}
