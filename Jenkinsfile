pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mikopbx:latest'
        CONTAINER_NAME = 'MikoPBX'
    }

    stages {
        stage('Checkout Repository') {
            steps {
                echo "Cloning GitHub repository..."
                git(
                    url: 'https://github.com/rashidmaqbool/zabbix-testing.git',
                    branch: 'main',
                    credentialsId: 'github-token'
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Deploy MikoPBX Container') {
            steps {
                echo "Deploying MikoPBX container..."
                sh """
                docker rm -f ${CONTAINER_NAME} || true
                docker run -d --name ${CONTAINER_NAME} -p 8080:80 ${DOCKER_IMAGE}
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "Verifying container is running..."
                sh "docker ps | grep ${CONTAINER_NAME}"
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed. Check logs!"
        }
    }
}
