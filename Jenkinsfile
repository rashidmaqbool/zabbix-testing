pipeline {
    agent any

    environment {
        COMPOSE_PATH = "${WORKSPACE}"
    }

    stages {
        stage('Prepare Environment') {
            steps {
                echo '🔹 Using checked-out workspace...'
                sh 'ls -l'
            }
        }

        stage('Pull Docker Images') {
            steps {
                echo '🔹 Pulling latest Zabbix Docker images...'
                sh '''
                cd ${COMPOSE_PATH}
                docker compose pull || true
                '''
            }
        }

        stage('Deploy Zabbix Stack') {
            steps {
                echo '🔹 Starting Zabbix stack using Docker Compose...'
                sh '''
                cd ${COMPOSE_PATH}
                docker compose up -d
                '''
            }
        }
stage('User Confirmation') {
            steps {
                script {
                    // Ask user to continue
                    def userInput = input(
                        id: 'Proceed', message: 'Do you want to continue?', parameters: [
                            [$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Yes/No']
                        ]
                    )
                    if (userInput) {
                        echo "User chose YES. Proceeding..."
                        // Add steps to run if Yes
                    } else {
                        echo "User chose NO. Stopping..."
                        // Optionally abort
                        currentBuild.result = 'ABORTED'
                        error("Build aborted by user")
                    }
                }
            }
        }

        stage('Next Step') {
            when {
                expression { return currentBuild.result != 'ABORTED' }
            }
            steps {
                echo "Running next stage..."
            }
        }
    }
        stage('Verify Deployment') {
            steps {
                echo '🔹 Checking running containers...'
                sh '''
                docker ps --filter "name=zabbix" --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Zabbix successfully deployed! Welldone Rashid you done it'
            echo 'Access the web interface at: http://<your-server-ip>:8080'
        }
        failure {
            echo '❌ Deployment failed — please check the Jenkins console logs for details.'
        }
    }
}