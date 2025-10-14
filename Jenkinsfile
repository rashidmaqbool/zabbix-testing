pipeline {
    agent any

    environment {
        COMPOSE_FILE = "docker-compose.yml"
        ENV_FILE = ".env"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://<your_token>@github.com/<your_username>/<your_repo>.git'
            }
        }

        stage('Prepare Environment') {
            steps {
                script {
                    sh '''
                    echo "🛠 Preparing environment..."

                    # Detect current user and group
                    ID_WWW_USER=$(id -u)
                    ID_WWW_GROUP=$(id -g)

                    # Generate .env file
                    cat <<EOF > .env
                    ID_WWW_USER=$ID_WWW_USER
                    ID_WWW_GROUP=$ID_WWW_GROUP
                    PBX_NAME=MikoPBX-in-Docker
                    SSH_PORT=23
                    WEB_PORT=8080
                    WEB_HTTPS_PORT=8443
                    EOF

                    echo "✅ Generated .env file:"
                    cat .env
                    '''
                }
            }
        }

        stage('Deploy MikoPBX') {
            steps {
                script {
                    sh '''
                    echo "🚀 Deploying MikoPBX..."
                    docker compose down || true
                    docker compose up -d
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "🔍 Verifying MikoPBX container..."
                docker ps --filter "name=mikopbx"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ MikoPBX deployed successfully!"
            echo "Access it at: https://<your_host_ip>:8443"
        }
        failure {
            echo "❌ Deployment failed. Check Jenkins console logs."
        }
    }
}
