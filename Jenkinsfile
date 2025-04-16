pipeline {
    agent any

    environment {
        // Application configuration
        IMAGE_NAME = 'karthikapadal23/flask-weather-app'
        DOCKER_REGISTRY = 'https://index.docker.io/v1/'
        
        // Credentials (must match your Jenkins credential IDs)
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        GIT_CREDENTIALS_ID = 'github-creds'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
                sh 'echo "Workspace cleaned successfully"'
            }
        }

        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    extensions: [
                        [$class: 'CleanBeforeCheckout'],
                        [$class: 'CloneOption', depth: 1, noTags: false, shallow: true],
                        [$class: 'RelativeTargetDirectory', relativeTargetDir: 'src']
                    ],
                    userRemoteConfigs: [[
                        url: 'https://github.com/karthikapadal23/Flask-Weather-App.git',
                        credentialsId: "${GIT_CREDENTIALS_ID}"
                    ]]
                ])
                
                // Verify the checkout worked
                sh '''
                    echo "Current directory:"
                    pwd
                    echo "Repository contents:"
                    ls -la src/
                    echo "Git status:"
                    cd src && git status && cd ..
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        dir('src') {
                            // Get commit hash for tagging
                            COMMIT_HASH = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                            
                            // Build Docker image
                            docker.build("${IMAGE_NAME}:latest")
                            docker.build("${IMAGE_NAME}:${COMMIT_HASH}")
                            
                            echo "Docker images built successfully"
                        }
                    } catch (Exception e) {
                        error("Docker build failed: ${e.getMessage()}")
                    }
                }
            }
        }

        stage('Test Application') {
            steps {
                script {
                    dir('src') {
                        try {
                            // Simple smoke test
                            sh 'docker run --rm ${IMAGE_NAME}:latest python --version'
                            sh 'docker run --rm ${IMAGE_NAME}:latest flask --version'
                            echo "Basic tests passed successfully"
                        } catch (Exception e) {
                            error("Tests failed: ${e.getMessage()}")
                        }
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        dir('src') {
                            sh '''
                                docker login -u $DOCKER_USER -p $DOCKER_PASS
                                docker push ${IMAGE_NAME}:latest
                                docker push ${IMAGE_NAME}:${COMMIT_HASH}
                                echo "Images pushed to Docker Hub successfully"
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed - cleaning up"
            script {
                // Clean up Docker containers and images
                sh 'docker system prune -f || true'
                
                // Archive important files
                archiveArtifacts artifacts: 'src/**', fingerprint: true
                
                // Clean workspace (optional)
                // cleanWs()
            }
        }
        success {
            echo "Pipeline succeeded! ${env.BUILD_URL}"
            // slackSend color: 'good', message: "SUCCESS: ${env.JOB_NAME} (#${env.BUILD_NUMBER})"
        }
        failure {
            echo "Pipeline failed! ${env.BUILD_URL}"
            // slackSend color: 'danger', message: "FAILURE: ${env.JOB_NAME} (#${env.BUILD_NUMBER})"
        }
    }
}
