pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/karthikapadal23/Flask-Weather-App.git'
        BRANCH = 'main'
        GIT_CREDENTIALS_ID = 'github-creds'
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
        WORKSPACE_DIR = 'app'
        IMAGE_NAME = 'karthikapadal23/weather-app' // Make sure this matches your Docker Hub repo name
    }

    stages {
        stage('Prepare Workspace') {
            steps {
                cleanWs()
                sh 'mkdir -p ${WORKSPACE_DIR}'
            }
        }

        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${BRANCH}"]],
                    extensions: [
                        [$class: 'CleanBeforeCheckout'],
                        [$class: 'RelativeTargetDirectory', relativeTargetDir: "${WORKSPACE_DIR}"],
                        [$class: 'CloneOption', depth: 1, noTags: true, shallow: true]
                    ],
                    userRemoteConfigs: [[
                        url: "${REPO_URL}",
                        credentialsId: "${GIT_CREDENTIALS_ID}"
                    ]]
                ])
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                dir("${WORKSPACE_DIR}") {
                    script {
                        docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                            sh """
                                docker build -t ${IMAGE_NAME}:latest .
                                docker push ${IMAGE_NAME}:latest
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed."
        }
        failure {
            echo "Pipeline failed - please check the logs."
        }
    }
}
