
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

                bat '''
                    if not exist index.html exit /b 1
                    if not exist style.css exit /b 1
                    if not exist script.js exit /b 1
                    if not exist Dockerfile exit /b 1

                    echo All files verified successfully
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'

                bat 'docker build -t %DOCKER_IMAGE%:latest .'
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
                    bat '''
                @echo off

                powershell.exe -NoProfile -NonInteractive -Command "$env:DOCKER_PASS | docker login --username $env:DOCKER_USER --password-stdin; if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }"

                if errorlevel 1 exit /b 1

                docker push %DOCKER_IMAGE%:latest

                if errorlevel 1 exit /b 1

                docker logout
            '''
                }
            }
        }

        stage('Deploy Website') {
            steps {
                echo 'Deploying website using Docker'

                bat '''
                    docker pull %DOCKER_IMAGE%:latest

                    if errorlevel 1 exit /b 1

                    docker rm -f my-website 2>NUL

                    docker run -d --name my-website -p 8081:80 %DOCKER_IMAGE%:latest

                    if errorlevel 1 exit /b 1
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
