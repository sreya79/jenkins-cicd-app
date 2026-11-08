pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    venv/bin/python -m unittest -v test_app.py
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t flask-cicd-app:${BUILD_NUMBER} .
                    docker tag flask-cicd-app:${BUILD_NUMBER} flask-cicd-app:latest
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    sh '''
                        echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin

                        docker tag flask-cicd-app:${BUILD_NUMBER} ${DOCKERHUB_USERNAME}/flask-cicd-app:${BUILD_NUMBER}
                        docker tag flask-cicd-app:latest ${DOCKERHUB_USERNAME}/flask-cicd-app:latest

                        docker push ${DOCKERHUB_USERNAME}/flask-cicd-app:${BUILD_NUMBER}
                        docker push ${DOCKERHUB_USERNAME}/flask-cicd-app:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    docker pull sreya79/flask-cicd-app:latest

                    docker rm -f flask-cicd-app || true

                    docker run -d \
                        --name flask-cicd-app \
                        -p 5000:5000 \
                        sreya79/flask-cicd-app:latest
                '''
            }
        }

    }
}
