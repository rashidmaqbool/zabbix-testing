pipeline {
  agent any

  environment {
    COMPOSE_PROJECT_NAME = 'mikopbx'
    // If you store DB password in Jenkins Credentials, uncomment and set credentialsId
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
        // If .env is not committed, copy example -> .env
        sh '''
        if [ ! -f .env ]; then
          if [ -f .env.example ]; then
            cp .env.example .env
            echo ".env created from .env.example. Please ensure secrets are set."
          else
            echo "ERROR: .env.example missing. Create .env or .env.example."
            exit 1
          fi
        else
          echo ".env already present"
        fi
        '''

        // Ensure host volume permissions (PUID/PGID values used in .env)
        sh '''
        # read PUID/PGID from .env (fallback to 1001 if not set)
        PUID=$(grep -E '^PUID=' .env | cut -d '=' -f2 || echo 1001)
        PGID=$(grep -E '^PGID=' .env | cut -d '=' -f2 || echo 1001)
        echo "Using PUID=$PUID PGID=$PGID for host directories"
        sudo mkdir -p /var/spool/mikopbx/cf /var/spool/mikopbx/storage || true
        sudo chown -R ${PUID}:${PGID} /var/spool/mikopbx || true
        '''
      }
    }

    stage('Deploy') {
      steps {
        // Use docker compose in workspace
        sh '''
        # Pull images (optional)
        docker compose pull || true

        # Teardown old containers and bring up new ones
        docker compose down --remove-orphans || true
        docker compose up -d
        '''

        // Wait & show basic status
        sh '''
        echo "Waiting 5s for containers to initialize..."
        sleep 5
        docker compose ps
        '''
      }
    }

    stage('Smoke Test (optional)') {
      steps {
        // Simple check to show logs if something failed
        sh '''
        # check if mikopbx container is running
        if [ "$(docker ps -q -f name=mikopbx)" = "" ]; then
          echo "mikopbx container not running. Dumping logs..."
          docker compose logs --no-color --tail=200
          exit 1
        else
          echo "mikopbx is running."
        fi
        '''
      }
    }
  }

  post {
    success {
      echo 'Deployment succeeded.'
    }
    failure {
      echo 'Deployment failed. See console output & container logs.'
    }
  }
}
