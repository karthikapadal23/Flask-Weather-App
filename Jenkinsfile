pipeline {
    agent any

    // Environment variables for easy configuration
    environment {
        REPO_URL = 'https://github.com/karthikapadal23/Flask-Weather-App.git'
        BRANCH = 'main'
        CREDENTIALS_ID = 'github-creds' // Must match your Jenkins credentials
        WORKSPACE_DIR = 'app' // Explicit workspace directory
    }

    stages {
        stage('Prepare Workspace') {
            steps {
                // Clean workspace and create target directory
                cleanWs()
                sh """
                    mkdir -p ${WORKSPACE_DIR}
                    echo "Workspace prepared at ${WORKSPACE_DIR}"
                """
            }
        }

        stage('Checkout Code') {
            steps {
                // Explicit checkout with all required parameters
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${BRANCH}"]],
                    extensions: [
                        [$class: 'CleanBeforeCheckout'],
                        [$class: 'RelativeTargetDirectory', relativeTargetDir: "${WORKSPACE_DIR}"],
                        [$class: 'CloneOption', timeout: 30, depth: 1, noTags: true, shallow: true]
                    ],
                    userRemoteConfigs: [[
                        url: "${REPO_URL}",
                        credentialsId: "${CREDENTIALS_ID}"
                    ]]
                ])

                // Verify the checkout
                sh """
                    echo "Current directory: \$(pwd)"
                    echo "Repository contents:"
                    ls -la ${WORKSPACE_DIR}/
                    cd ${WORKSPACE_DIR} && git status && git log -1 --oneline
                """
            }
        }

        stage('Build and Deploy') {
            steps {
                dir("${WORKSPACE_DIR}") {
                    // Your build and deployment steps here
                    sh 'echo "Build would execute here"'
                    // Example:
                    // sh 'docker build -t myapp .'
                    // sh 'docker push myapp'
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed"
            // archiveArtifacts artifacts: "${WORKSPACE_DIR}/**", fingerprint: true
        }
        failure {
            echo "Pipeline failed - check the logs"
            // Add notification steps here if needed
        }
    }
}
