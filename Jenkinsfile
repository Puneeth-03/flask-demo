pipeline {
    agent any

    stages {

        stage('Test') {
            agent {
                docker {
                    image 'python:3.12-slim'
                    reuseNode true
                }
            }

            environment {
                HOME = "${WORKSPACE}"
            }

            steps {
                sh '''
                    echo "Python:"
                    python --version

                    echo "Installing dependencies..."
                    python -m pip install --user --no-cache-dir -r requirements.txt

                    echo "Running tests..."
                    python -m pytest -v
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Flask application image..."
                    docker build -t flask-demo:${BUILD_NUMBER} .
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