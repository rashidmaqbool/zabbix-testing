pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        echo '🔹 Cloning public GitHub repository...'
        // Since it's a public repo, no credentials needed
        checkout([$class: 'GitSCM',
                  branches: [[name: '*/main']],
                  userRemoteConfigs: [[url: 'https://github.com/rashidmaqbool/zabbix-testing.git']]
        ])
      }
    }

    stage('Pull Docker Images') {
      steps {
        echo '🔹 Pulling latest Zabbix images...'
        sh 'docker compose pull || true'
      }
    }

    stage('Deploy Zabbix Stack') {
      steps {
        echo '🔹 Deploying Zabbix stack...'
        sh 'docker compose up -d'
      }
    }

    stage('Verify Deployment') {
      steps {
        echo '🔹 Verifying running containers...'
        sh 'docker ps --format "table {{.Names}}\t{{.Status}}"'
      }
    }
  }

  post {
    success {
      echo '✅ Zabbix successfully deployed!'
      echo 'Access web interface at: http://<server-ip>:8080'
    }
    failure {
      echo '❌ Deployment failed. Check the Jenkins console for logs.'
    }
  }
}

