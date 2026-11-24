
pipeline {
    agent any

    environment {
        DOCKER_USER = 'sasimaddela'
        IMAGE_NAME = 'sasi'
        CREDENTIALS_ID = 'dockerhub-credentials'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo 'Checking out source code from GitHub'

                git branch: 'main',
                    url: 'https://github.com/maddelasasi/sasi.git'
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                    echo "Checking project files..."
                    ls -la
                    test -f Dockerfile
                    test -f app.py
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKER_USER}/${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${CREDENTIALS_ID}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USERNAME" --password-stdin
                    '''
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh '''
                    docker push ${DOCKER_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
                    docker push ${DOCKER_USER}/${IMAGE_NAME}:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Docker image pushed to Docker Hub!'
        }

        failure {
            echo 'FAILURE: Check the Jenkins Console Output.'
        }

        always {
            sh '''
                docker logout || true
                docker image prune -f || true
            '''
        }
    }
}

