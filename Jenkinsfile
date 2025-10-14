pipeline {
    agent any

    environment {
        COMPOSE_FILE = "docker-compose.yml"
        ENV_FILE = ".env"
    }

    stages {

        stage('Prepare Environment') {
            steps {
                script {
                    sh '''
                    echo "🛠 Preparing environment..."

                    ID_WWW_USER=$(id -u)
                    ID_WWW_GROUP=$(id -g)

                    cat <<EOF > .env
                    ID_WWW_USER=$ID_WWW_USER
                    ID_WWW_GROUP=$ID_WWW_GROUP
                    PROJECT_NAME=Zabbix-in-Docker
                    DB_PORT=5432
                    WEB_PORT=8080
                    EOF

                    echo "✅ Generated .env file:"
                    cat .env
                    '''
                }
            }
        }

        stage('Deploy Zabbix') {
            steps {
                script {
                    sh '''
                    echo "🚀 Deploying Zabbix..."
                    docker compose down || true
                    docker compose up -d
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                echo "🔍 Verifying Zabbix container..."
                docker ps --filter "name=zabbix"
                '''
            }
        }
    }

    post {
        success {
            script {
                def ip = sh(script: "hostname -I | awk '{print \$1}'", returnStdout: true).trim()
                echo "✅ Zabbix deployed successfully!"
                echo "Access it at: http://${ip}:8080"
            }
        }
        failure {
            echo "❌ Deployment failed. Check Jenkins console logs."
        }
    }
}
