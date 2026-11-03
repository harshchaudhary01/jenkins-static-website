
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'harshchaudhary01/my-static-website'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Downloading source code from GitHub'
                checkout scm
            }
        }

        stage('Test') {
            steps {
                echo 'Checking website files'

                sh '''
                    test -f index.html
                    test -f style.css
                    test -f script.js
                    test -f Dockerfile

                    echo "All files verified successfully"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'

                sh '''
                    docker build -t ${DOCKER_IMAGE}:latest .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                        -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy Website') {
            steps {
                sh '''
                    docker pull ${DOCKER_IMAGE}:latest

                    docker rm -f my-website || true

                    docker run -d \
                    --name my-website \
                    -p 8081:80 \
                    ${DOCKER_IMAGE}:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'Website deployed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check console output.'
        }
    }
}
