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

    stage('Prepare Environment') {
      steps {
        echo "⚙️ Preparing environment variables..."
        sh '''
          echo "Current environment file:"
          cat .env || echo ".env not found!"

          # Ensure valid RTP range to avoid port conflicts
          sed -i 's/^RTP_START=.*/RTP_START=20000/' .env
          sed -i 's/^RTP_END=.*/RTP_END=21000/' .env

          echo "✅ Updated .env file:"
          cat .env
        '''
      }
    }

    stage('Cleanup old containers') {
      steps {
        echo "🧹 Cleaning up old containers..."
        sh '''
          docker compose down --remove-orphans || true
          docker container prune -f || true
          docker network prune -f || true
        '''
      }
    }

    stage('Deploy MikoPBX') {
      steps {
        echo "🚀 Deploying MikoPBX..."
        sh '''
          docker compose pull || true
          docker compose up -d
          echo "⏳ Waiting 10 seconds for containers to initialize..."
          sleep 10
          docker compose ps
        '''
      }
    }

    stage('Health Check') {
      steps {
        echo "🩺 Checking if MikoPBX is running..."
        sh '''
          if [ "$(docker ps -q -f name=mikopbx_app)" = "" ]; then
            echo "❌ MikoPBX container not running. Dumping logs..."
            docker compose logs --no-color --tail=100
            exit 1
          else
            echo "✅ MikoPBX container is running successfully."
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
