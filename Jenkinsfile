pipeline {
    agent any

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/jibishwaran07/python-ci-cd-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                  python3 -m pip install --user -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                  python3 -m pytest
                '''
            }
        }

        stage('Build & Push Docker Image (ONLY v1 v2 v3)') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      set -e

                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                      # Pull existing images if they exist (safe on first run)
                      docker pull jibishwaran07/python-ci-cd-app:v1 || true
                      docker pull jibishwaran07/python-ci-cd-app:v2 || true
                      docker pull jibishwaran07/python-ci-cd-app:v3 || true

                      # Rotate tags
                      docker tag jibishwaran07/python-ci-cd-app:v2 jibishwaran07/python-ci-cd-app:v3 || true
                      docker tag jibishwaran07/python-ci-cd-app:v1 jibishwaran07/python-ci-cd-app:v2 || true

                      # Build new image as v1
                      docker build -t jibishwaran07/python-ci-cd-app:v1 .

                      # Push only 3 tags (NO latest)
                      docker push jibishwaran07/python-ci-cd-app:v1
                      docker push jibishwaran07/python-ci-cd-app:v2 || true
                      docker push jibishwaran07/python-ci-cd-app:v3 || true
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                      kubectl set image deployment/python-ci-cd-app \
                      flask-app=jibishwaran07/python-ci-cd-app:v1

                      kubectl rollout status deployment/python-ci-cd-app
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo "Deployment failed – rolling back"
            sh '''
              kubectl rollout undo deployment/python-ci-cd-app
            '''
        }
    }
}
