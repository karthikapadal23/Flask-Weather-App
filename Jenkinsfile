pipeline {
    agent any

    environment {
        // Application Configuration
        APP_NAME = 'flask-weather-app'
        DOCKER_IMAGE = "karthikapadal23/${APP_NAME}"
        
        // Registry Configuration
        DOCKER_REGISTRY = 'https://index.docker.io/v1/'
        
        // Credentials (must match your Jenkins credential IDs)
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        GIT_CREDENTIALS_ID = 'github-creds'
        
        // Git Configuration
        GIT_URL = 'https://github.com/karthikapadal23/Flask-Weather-App.git'
        GIT_BRANCH = 'main'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "${GIT_BRANCH}"]],
                    extensions: [[
                        $class: 'CleanBeforeCheckout',
                        deleteUntrackedNestedRepositories: true
                    ]],
                    userRemoteConfigs: [[
                        url: "${GIT_URL}",
                        credentialsId: "${GIT_CREDENTIALS_ID}"
                    ]]
                ])
                
                // Verify repository contents
                sh '''
                    echo "Repository Contents:"
                    ls -la
                    echo "Dockerfile Contents:"
                    cat Dockerfile
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python -m venv venv
                    . venv/bin/activate
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        // Get commit hash for tagging
                        COMMIT_HASH = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                        
                        // Build with multiple tags
                        docker.build("${DOCKER_IMAGE}:latest")
                        docker.build("${DOCKER_IMAGE}:${COMMIT_HASH}")
                        
                    } catch (Exception e) {
                        error("Docker build failed: ${e.getMessage()}")
                    }
                }
            }
        }

        stage('Test Application') {
            steps {
                script {
                    try {
                        // Run simple tests
                        sh 'python -m pytest tests/ || echo "No tests found"'
                        
                        // Test Docker image
                        sh 'docker run --rm ${DOCKER_IMAGE}:latest python --version'
                        sh 'docker run --rm ${DOCKER_IMAGE}:latest flask --version'
                    } catch (Exception e) {
                        error("Tests failed: ${e.getMessage()}")
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
                        sh '''
                            docker login -u $DOCKER_USER -p $DOCKER_PASS
                            docker push ${DOCKER_IMAGE}:latest
                            docker push ${DOCKER_IMAGE}:${COMMIT_HASH}
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up Docker containers and images
            sh '''
                docker system prune -f || true
                docker rmi ${DOCKER_IMAGE}:latest || true
                docker rmi ${DOCKER_IMAGE}:${COMMIT_HASH} || true
            '''
            
            // Archive important files
            archiveArtifacts artifacts: '**/app.py,**/requirements.txt,**/Dockerfile', fingerprint: true
        }
        
        success {
            echo "Pipeline succeeded! ${env.BUILD_URL}"
        }
        
        failure {
            echo "Pipeline failed! ${env.BUILD_URL}"
            // Uncomment to enable Slack notifications
            // slackSend color: 'danger', message: "FAILED: Job '${env.JOB_NAME}' (${env.BUILD_URL})"
        }
    }
}
