pipeline {
    agent any

    environment {
        IMAGE_NAME = "jibishwaran07/python-ci-cd-app"
        IMAGE_TAG  = "latest"
        DOCKER_CREDS = credentials('dockerhub-creds')
        KUBECONFIG_CRED = credentials('kubeconfig')
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/jibishwaran07/python-ci-cd-app.git'
            }
        }

        stage('Install Python Dependencies') {
            steps {
                sh '''
                  python3 -m pip install --user -r requirements.txt
                '''
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh '''
                  python3 -m pytest
                '''
            }
        }

          

stage('Build & Push Docker Image (v1 v2 v3 only)') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'dockerhub-creds',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {
            sh '''
              echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

              # Pull existing tags (ignore if not present)
              docker pull jibishwaran07/python-ci-cd-app:v1 || true
              docker pull jibishwaran07/python-ci-cd-app:v2 || true
              docker pull jibishwaran07/python-ci-cd-app:v3 || true

              # Rotate tags
              docker tag jibishwaran07/python-ci-cd-app:v2 jibishwaran07/python-ci-cd-app:v3 || true
              docker tag jibishwaran07/python-ci-cd-app:v1 jibishwaran07/python-ci-cd-app:v2 || true

              # Build new image as v1
              docker build -t jibishwaran07/python-ci-cd-app:v1 .

              # Push ONLY v1 v2 v3 (NO latest)
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
              flask-app=$IMAGE_NAME:$IMAGE_TAG

              kubectl rollout status deployment/python-ci-cd-app
            '''
        }
    }
}






        stage('Docker Login') {
            steps {
                sh '''
                  echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                  docker push $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                      kubectl apply -f k8s/
                      kubectl rollout restart deployment python-ci-cd-app
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD Pipeline completed successfully'
        }
        failure {
            echo '❌ CI/CD Pipeline failed'
        }
    }
}




post {
    failure {
        echo "Deployment failed – rolling back to previous version"
        sh """
          kubectl rollout undo deployment/python-ci-cd-app
        """
    }
}




post {
    success {
        echo '✅ Deployment successful'
    }

    failure {
        echo '❌ Deployment failed – rolling back'
        sh '''
          kubectl rollout undo deployment/python-ci-cd-app
        '''
    }
}


stage('Deploy to Kubernetes') {
    steps {
        withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
            sh '''
              kubectl annotate deployment/python-ci-cd-app \
              kubernetes.io/change-cause="Deploy image ${IMAGE_NAME}:${IMAGE_TAG}" --overwrite

              kubectl set image deployment/python-ci-cd-app \
              flask-app=$IMAGE_NAME:$IMAGE_TAG

              kubectl rollout status deployment/python-ci-cd-app
            '''
        }
    }
}
