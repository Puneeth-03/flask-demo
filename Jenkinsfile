pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'python:3.12-slim'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    python --version
                    pip install --no-cache-dir -r requirements.txt
                    python -m pytest -v
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t flask-demo:${BUILD_NUMBER} .
                '''
            }
        }
    }

    post {
        success {
            echo 'CI pipeline completed successfully.'
        }

        failure {
            echo 'CI pipeline failed.'
        }
    }
}