pipeline {
    agent any

    environment {
        IMAGE_NAME = 'karthikapadal23/flask-weather-app'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // Jenkins credentials ID for Docker Hub
        GIT_CREDENTIALS_ID = 'github-creds' // Jenkins credentials ID for GitHub
    }

    stages {
        stage('Clone Repo') {
            steps {
                git credentialsId: "${GIT_CREDENTIALS_ID}", url: 'https://github.com/karthikapadal23/Flask-Weather-App.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:latest")
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKERHUB_CREDENTIALS}") {
                        echo 'Logged in to DockerHub'
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', "${DOCKERHUB_CREDENTIALS}") {
                        docker.image("${IMAGE_NAME}:latest").push()
                    }
                }
            }
        }
    }
}
