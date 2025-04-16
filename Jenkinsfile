pipeline {
    agent any

    environment {
        // Docker configuration
        IMAGE_NAME = 'karthikapadal23/flask-weather-app'
        DOCKER_REGISTRY = 'https://index.docker.io/v1/'
        
        // Credentials (defined in Jenkins Credentials Manager)
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        GIT_CREDENTIALS_ID = 'github-creds'  // Make sure this matches your Jenkins credentials ID
        
        // Git configuration
        GIT_REPO = 'https://github.com/karthikapadal23/Flask-Weather-App.git'
        GIT_BRANCH = 'main'  // or your preferred branch
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "${GIT_BRANCH}"]],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: "${GIT_CREDENTIALS_ID}",
                        url: "${GIT_REPO}"
                    ]]
                ])
                
                // Verify checkout
                sh 'git branch'
                sh 'ls -la'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        // Build with cache and proper tagging
                        dockerImage = docker.build(
                            "${IMAGE_NAME}:latest",
                            "--build-arg BUILD_DATE=`date -u +\"%Y-%m-%dT%H:%M:%SZ\"` ."
                        )
                    } catch (Exception e) {
                        error("Docker build failed: ${e.getMessage()}")
                    }
                }
            }
        }

        stage('Test Docker Image') {
            steps {
                script {
                    // Add any tests for your Docker image here
                    sh 'docker run --rm ${IMAGE_NAME}:latest python --version'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    try {
                        docker.withRegistry("${DOCKER_REGISTRY}", "${DOCKERHUB_CREDENTIALS}") {
                            dockerImage.push('latest')
                            
                            // Optionally push with git commit hash as tag
                            COMMIT_HASH = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                            dockerImage.push("${COMMIT_HASH}")
                        }
                    } catch (Exception e) {
                        error("Docker push failed: ${e.getMessage()}")
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up Docker images to save disk space
            script {
                sh 'docker system prune -af || true'
            }
            
            // Clean workspace (optional)
            // cleanWs()
        }
        
        success {
            echo 'Pipeline completed successfully!'
            slackSend(color: 'good', message: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        }
        
        failure {
            echo 'Pipeline failed!'
            slackSend(color: 'danger', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'")
        }
    }
}
