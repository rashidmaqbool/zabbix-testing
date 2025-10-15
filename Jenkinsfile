pipeline {
    agent any

    stages {
     

        stage('Deploy MikoPBX Container') {
            steps {
                echo 'Deploying MikoPBX using Docker Compose with .env file...'
                sh '''
                if [ ! -f .env ]; then
                    echo ".env file not found! Deployment aborted."
                    exit 1
                fi
                docker-compose down
                docker-compose up -d
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Listing running containers...'
                sh 'docker ps'
            }
        }
    }

    post {
        success {
            echo 'MikoPBX deployed successfully!'
        }
        failure {
            echo 'Deployment failed. Check logs!'
        }
    }
}
