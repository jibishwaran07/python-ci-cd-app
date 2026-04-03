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

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
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
