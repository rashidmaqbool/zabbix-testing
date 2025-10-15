pipeline {
    agent any

    environment {
        MIKO_DIR = "C:/mikopbx"
        GIT_REPO = "https://github.com/rashidmaqbool/zabbix-testing.git"
        BRANCH = "main"
    }

    stages {
        stage('Prepare Directory') {
            steps {
                script {
                    // Create mikopbx folder if it doesn't exist
                    bat """
                    if not exist "%MIKO_DIR%" mkdir "%MIKO_DIR%"
                    """
                }
            }
        }

        stage('Clone GitHub Repository') {
            steps {
                script {
                    // Clone the repo into mikopbx folder or pull latest changes
                    bat """
                    cd "%MIKO_DIR%"
                    if exist ".git" (
                        echo Repository already exists. Pulling latest changes...
                        git reset --hard
                        git pull origin %BRANCH%
                    ) else (
                        git clone -b %BRANCH% %GIT_REPO% .
                    )
                    """
                }
            }
        }

        stage('Run Docker Compose') {
            steps {
                script {
                    // Start the MikoPBX container
                    bat """
                    cd "%MIKO_DIR%"
                    docker compose down
                    docker compose up -d
                    """
                }
            }
        }
    }

    post {
        success {
            echo "MikoPBX Docker container is up and running!"
        }
        failure {
            echo "Pipeline failed. Check Jenkins logs for errors."
        }
    }
}
