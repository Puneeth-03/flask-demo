pipeline {
    agent any

    environment {
        IMAGE_NAME = 'ghcr.io/puneeth-03/flask-demo'
        GITOPS_REPO = 'https://github.com/Puneeth-03/flask-demo-gitops.git'
    }

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
                    docker build \
                        -t ${IMAGE_NAME}:${BUILD_NUMBER} \
                        -t ${IMAGE_NAME}:latest \
                        .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-ghcr',
                        usernameVariable: 'GH_USERNAME',
                        passwordVariable: 'GH_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$GH_TOKEN" | docker login ghcr.io \
                            -u "$GH_USERNAME" \
                            --password-stdin

                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest

                        docker logout ghcr.io
                    '''
                }
            }
        }

        stage('Update GitOps Repository') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-gitops',
                        usernameVariable: 'GH_USERNAME',
                        passwordVariable: 'GH_TOKEN'
                    )
                ]) {
                    sh '''
                        rm -rf gitops

                        git clone \
                            https://${GH_USERNAME}:${GH_TOKEN}@github.com/Puneeth-03/flask-demo-gitops.git \
                            gitops

                        cd gitops

                        git config user.name "jenkins"
                        git config user.email "jenkins@localhost"

                        sed -i \
                            "s#image: ghcr.io/puneeth-03/flask-demo:.*#image: ghcr.io/puneeth-03/flask-demo:${BUILD_NUMBER}#" \
                            deployment.yaml

                        git add deployment.yaml

                        git commit -m "Deploy flask-demo ${BUILD_NUMBER}" || true

                        git push origin main
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully.'
        }

        failure {
            echo 'CI/CD pipeline failed.'
        }
    }
}