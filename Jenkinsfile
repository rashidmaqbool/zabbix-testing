pipeline {
  agent any

  environment {
    COMPOSE_PROJECT_NAME = 'zabbix'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Clean and Generate .env') {
      steps {
        sh '''
        echo "🧹 Cleaning any previous .env file..."
        rm -f .env

        echo "📝 Generating fresh .env file for Zabbix..."
        cat > .env <<EOF
POSTGRES_USER=zabbix
POSTGRES_PASSWORD=zabbix_pass
POSTGRES_DB=zabbix
MYSQL_USER=zabbix
MYSQL_PASSWORD=zabbix_pass
MYSQL_ROOT_PASSWORD=root_pass
DB_SERVER_HOST=mysql-server
DB_SERVER_PORT=3306
ZBX_SERVER_NAME=MyZabbix
EOF

        echo "✅ .env file generated successfully."
        echo "Content of .env:"
        cat .env
        '''
      }
    }

    stage('Deploy Zabbix') {
      steps {
        sh '''
        echo "🚀 Deploying Zabbix..."
        docker compose down || true
        docker compose up -d
        echo "✅ Deployment complete."
        docker compose ps
        '''
      }
    }

    stage('Verify Containers') {
      steps {
        sh '''
        echo "🔍 Checking Zabbix container status..."
        if [ "$(docker ps -q -f name=zabbix)" = "" ]; then
          echo "❌ Zabbix container not running. Dumping logs..."
          docker compose logs --no-color --tail=200
          exit 1
        else
          echo "✅ Zabbix is running successfully!"
        fi
        '''
      }
    }
  }

  post {
    success {
      echo '✅ Jenkins pipeline completed successfully!'
    }
    failure {
      echo '❌ Jenkins pipeline failed. Check console output for details.'
    }
  }
}
