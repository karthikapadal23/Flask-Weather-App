pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // You must set this in Jenkins > Credentials
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/karthikapadal23/Flask-Weather-App.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("karthikapadal23/flask-weather-app")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                        dockerImage.push('latest')
                    }
                }
            }
        }
    }

    post {
        failure {
            echo '❌ Pipeline failed. Check error logs.'
        }
        success {
            echo '✅ Pipeline completed successfully.'
        }
    }
}
