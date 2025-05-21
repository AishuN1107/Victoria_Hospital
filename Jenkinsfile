pipeline {
  agent any

  environment {
    SONARQUBE = 'MySonarQube'
  }

  stages {

    stage('Build') {
      steps {
        echo 'Installing dependencies...'
        bat 'npm install'
      }
    }

    stage('Test') {
      steps {
        echo 'Running unit tests...'
        bat 'npm test'
      }
    }

    stage('Code Quality') {
      steps {
        echo 'Running SonarQube analysis...'
        withSonarQubeEnv(SONARQUBE) {
          bat 'sonar-scanner'
        }
      }
    }

    stage('Security Scan') {
      environment {
        SNYK_TOKEN = credentials('snyk-token')
      }
      steps {
        echo 'Running Snyk security scan...'
        bat '''
          npm install -g snyk
          snyk auth %SNYK_TOKEN%
          snyk test || exit 0
        '''
      }
    }

    stage('Deploy to Test Environment') {
      steps {
        echo 'Deploying with Docker Compose...'
        bat 'docker-compose down || exit 0'
        bat 'docker-compose up -d --build'
      }
    }

    stage('Release') {
      steps {
        echo 'Creating release tag...'
        bat '''
          git config --global user.email "naidusaiaishu2003@gmail.com"
          git config --global user.name "AishuN1107"
          git tag -a v1.0.%BUILD_NUMBER% -m "Release v1.0.%BUILD_NUMBER%"
          git push origin v1.0.%BUILD_NUMBER%
        '''
      }
    }

    stage('Monitoring & Alerts') {
      steps {
        echo 'Monitoring health check...'
        bat 'curl -f http://localhost:3000/health || echo Health check failed'
      }
    }
  }
}
