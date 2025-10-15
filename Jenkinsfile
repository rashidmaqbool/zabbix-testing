pipeline {
  agent {
    docker {
      image 'docker:27.1.1-dind'
      args '--privileged -v /var/run/docker.sock:/var/run/docker.sock -u root'
    }
  }

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

          sed -i '/^$/d' .env
          sed -i '/^#/d' .env
          sed -i '/echo/d' .env
          sed -i '/EOF/d' .env
          sed -i '/cat/d' .env

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
