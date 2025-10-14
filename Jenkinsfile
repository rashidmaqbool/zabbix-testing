pipeline {
    agent any

    environment {
        COMPOSE_FILE = "docker-compose.yml"
        ENV_FILE = ".env"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/yourusername/miko-pbx.git'
            }
        }

        stage('Prepare Environment') {
            steps {
                script {
                    sh '''
                    # Detect current user and group
                    export ID_WWW_USER=$(id -u)
                    export ID_WWW_GROUP=$(id -g)
                    
                    # Generate fresh .env file
                    echo "ID_WWW_USER=$ID_WWW_USER" > .env
                    echo "ID_WWW_GROUP=$ID_WWW_GROUP" >> .env
                    echo "PBX_NAME=MikoPBX-in-Docker" >> .env
                    echo "SSH_PORT=23" >> .env
                    echo "WEB_PORT=8080" >> .env
                    echo "WEB_HTTPS_PORT=8443" >> .env
                    
                    cat .env
                    '''
                }
            }
        }

        stage('Deploy MikoPBX') {
            steps {
                sh '''
                echo "🧩 Bringing up MikoPBX container..."
                docker compose down || true
                docker compose up -d
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "🔍 Verifying MikoPBX container status..."
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
