pipeline {
  agent any

  environment {
    COMPOSE_PROJECT_NAME = 'mikopbx'
    // Uncomment if you want to pull DB password from Jenkins credentials:
    // DB_PASSWORD = credentials('jenkins-db-password-id')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Prepare .env') {
      steps {
        script {
          // Safely generate or validate .env file
          sh '''
          if [ ! -f .env ]; then
            echo "No .env found — generating a new one..."
            cat > .env <<EOF
PUID=1001
PGID=1001
TZ=Asia/Karachi
MIKO_IMAGE=mikopbx:latest
DB_IMAGE=postgres:14
DB_HOST=db
DB_PORT=5432
DB_USER=mikopbx
DB_PASSWORD=StrongPassword123
DB_NAME=mikopbx
WEB_PORT=8080
SIP_PORT=5060
RTP_START=10000
RTP_END=20000
COMPOSE_PROJECT_NAME=mikopbx
EOF
            echo ".env file created successfully."
          else
            echo ".env already exists. Skipping creation."
          fi
          '''

          // Ensure host directories exist and permissions are correct
          sh '''
          PUID=$(grep -E '^PUID=' .env | cut -d '=' -f2 || echo 1001)
          PGID=$(grep -E '^PGID=' .env | cut -d '=' -f2 || echo 1001)
          echo "Ensuring directories exist with correct ownership..."
          sudo mkdir -p /var/spool/mikopbx/cf /var/spool/mikopbx/storage || true
          sudo chown -R ${PUID}:${PGID} /var/spool/mikopbx || true
          '''
        }
      }
    }

    stage('Deploy') {
      steps {
        sh '''
        echo "🚀 Deploying MikoPBX..."
        docker compose pull || true
        docker compose down --remove-orphans || true
        docker compose up -d
        echo "✅ Deployment complete."
        docker compose ps
        '''
      }
    }

    stage('Smoke Test (optional)') {
      steps {
        sh '''
        echo "Checking if mikopbx container is running..."
        if [ "$(docker ps -q -f name=mikopbx)" = "" ]; then
          echo "❌ mikopbx container not running. Dumping logs..."
          docker compose logs --no-color --tail=200
          exit 1
        else
          echo "✅ mikopbx is running successfully."
        fi
        '''
      }
    }
  }

  post {
    success {
      echo '✅ Deployment succeeded!'
    }
    failure {
      echo '❌ Deployment failed. See console logs for details.'
    }
  }
}
