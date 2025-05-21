pipeline {
  agent any

  environment {
    SONARQUBE = 'MySonarQube'
  }

  stages {

    stage('Build') {
      steps {
        echo 'Installing dependencies...'
        sh '/bin/sh -c "npm install"'
      }
    }

    stage('Test') {
      steps {
        echo 'Running unit tests...'
        sh '/bin/sh -c "npm test"'
      }
    }

    stage('Code Quality') {
      steps {
        echo 'Running SonarQube analysis...'
        withSonarQubeEnv(SONARQUBE) {
          sh '/bin/sh -c "sonar-scanner"'
        }
      }
    }

    stage('Security Scan') {
      environment {
        SNYK_TOKEN = credentials('snyk-token')
      }
      steps {
        echo 'Running Snyk security scan...'
        sh '''
          /bin/sh -c "
            npm install -g snyk
            snyk auth ${SNYK_TOKEN}
            snyk test || true
          "
        '''
      }
    }



    stage('Deploy to Test Environment') {
      steps {
        echo 'Deploying with Docker Compose...'
        sh 'docker-compose down || true'
        sh 'docker-compose up -d --build'
      }
    }

    stage('Release') {
      steps {
        echo 'Creating release tag...'
        sh '''
          git config --global user.email "naidusaiaishu2003@gmail.com"
          git config --global user.name "AishuN1107"
          git tag -a v1.0.${BUILD_NUMBER} -m "Release v1.0.${BUILD_NUMBER}"
          git push origin v1.0.${BUILD_NUMBER}
        '''
      }
    }

    stage('Monitoring & Alerts') {
      steps {
        echo 'Monitoring health check...'
        sh 'curl -f http://localhost:3000/health || echo "Health check failed"'
      }
    }
  }
}
