pipeline {
  agent any

  environment {
    COMPOSE_PROJECT_NAME = 'mikopbx'
  }

  stages {

    stage('Checkout') {
      steps {
        echo "📦 Checking out source..."
        checkout scm
      }
    }

    stage('Prepare .env') {
      steps {
        echo "⚙️ Preparing .env file..."
        sh '''
          # If .env doesn’t exist, create a new clean one
          if [ ! -f .env ]; then
            echo "Creating fresh .env file..."
            cat <<EOF > .env
ID_WWW_USER=0
ID_WWW_GROUP=0
PROJECT_NAME=Zabbix-in-Docker
DB_PORT=5432
WEB_PORT=8080
EOF
          fi

          # Remove any invalid lines (just in case)
          sed -i '/^$/d' .env                # Remove empty lines
          sed -i '/^#/d' .env                # Remove comments
          sed -i '/echo/d' .env              # Remove echo lines
          sed -i '/EOF/d' .env               # Remove stray EOF
          sed -i '/cat/d' .env               # Remove cat commands

          echo "✅ Cleaned .env file:"
          cat .env
        '''
      }
    }

    stage('Deploy') {
      steps {
        echo "🚀 Deploying Zabbix..."
        sh '''
          docker compose pull || true
          docker compose down --remove-orphans || true
          docker compose up -d

          echo "⏳ Waiting 5 seconds for containers to initialize..."
          sleep 5
          docker compose ps
        '''
      }
    }

    stage('Health Check') {
      steps {
        echo "🩺 Checking if container is running..."
        sh '''
          if [ "$(docker ps -q -f name=zabbix)" = "" ]; then
            echo "❌ Zabbix container not running. Dumping logs..."
            docker compose logs --no-color --tail=100
            exit 1
          else
            echo "✅ Zabbix container is running successfully."
          fi
        '''
      }
    }
  }

  post {
    success {
      echo '🎉 Deployment succeeded!'
    }
    failure {
      echo '❌ Deployment failed! Check Jenkins logs and Docker Compose output.'
    }
  }
}
