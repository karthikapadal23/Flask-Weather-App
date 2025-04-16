pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/karthikapadal23/Flask-Weather-App.git'
        BRANCH = 'main'
        CREDENTIALS_ID = 'github-creds' // Your GitHub credentials ID in Jenkins
        WORKSPACE_DIR = 'app' // Target folder for the cloned repo
    }

    stages {
        stage('Prepare Workspace') {
            steps {
                cleanWs()
                sh """
                    mkdir -p ${WORKSPACE_DIR}
                    echo "✅ Workspace prepared at ${WORKSPACE_DIR}"
                """
            }
        }

        stage('Checkout Code') {
            steps {
                // Perform the Git checkout explicitly
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

                // Verify repo status
                sh """
                    echo "🔍 Verifying Git checkout..."
                    cd ${WORKSPACE_DIR}
                    if [ -d .git ]; then
                        echo "✅ .git directory exists"
                        git status
                        git log -1 --oneline
                    else
                        echo "❌ Git repository not found!"
                        exit 1
                    fi
                """
            }
        }

        stage('Build and Deploy') {
            steps {
                dir("${WORKSPACE_DIR}") {
                    // Replace this with your actual build commands
                    sh 'echo "🔧 Build would execute here..."'
                    // e.g., docker build, docker push, etc.
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Pipeline execution completed"
        }
        failure {
            echo "❌ Pipeline failed - please check the logs"
        }
    }
}
