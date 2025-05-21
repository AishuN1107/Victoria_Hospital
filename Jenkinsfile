pipeline {
  agent any

  environment {
    SONARQUBE = 'MySonarQube'
  }

  stages {

    stage('Build') {
      steps {
        echo 'Installing dependencies...'
        script {
          def proc = ['/bin/bash', '-c', 'npm install'].execute()
          proc.waitForProcessOutput(System.out, System.err)
        }
      }
    }

    stage('Test') {
      steps {
        echo 'Running unit tests...'
        script {
          def proc = ['/bin/bash', '-c', 'npm test'].execute()
          proc.waitForProcessOutput(System.out, System.err)
        }
      }
    }

    stage('Code Quality') {
      steps {
        echo 'Running SonarQube analysis...'
        withSonarQubeEnv(SONARQUBE) {
          script {
            def proc = ['/bin/bash', '-c', 'sonar-scanner'].execute()
            proc.waitForProcessOutput(System.out, System.err)
          }
        }
      }
    }

    stage('Security Scan') {
      environment {
        SNYK_TOKEN = credentials('snyk-token')
      }
      steps {
        echo 'Running Snyk security scan...'
        script {
          def cmd = '''
            npm install -g snyk &&
            snyk auth ${SNYK_TOKEN} &&
            snyk test || true
          '''
          def proc = ['/bin/bash', '-c', cmd].execute()
          proc.waitForProcessOutput(System.out, System.err)
        }
      }
    }

    stage('Deploy to Test Environment') {
      steps {
        echo 'Deploying with Docker Compose...'
        script {
          def proc = ['/bin/bash', '-c', 'docker-compose down || true && docker-compose up -d --build'].execute()
          proc.waitForProcessOutput(System.out, System.err)
        }
      }
    }

    stage('Release') {
      steps {
        echo 'Creating release tag...'
        script {
          def cmd = '''
            git config --global user.email "naidusaiaishu2003@gmail.com"
            git config --global user.name "AishuN1107"
            git tag -a v1.0.${BUILD_NUMBER} -m "Release v1.0.${BUILD_NUMBER}"
            git push origin v1.0.${BUILD_NUMBER}
          '''
          def proc = ['/bin/bash', '-c', cmd].execute()
          proc.waitForProcessOutput(System.out, System.err)
        }
      }
    }

    stage('Monitoring & Alerts') {
      steps {
        echo 'Monitoring health check...'
        script {
          def proc = ['/bin/bash', '-c', 'curl -f http://localhost:3000/health || echo "Health check failed"'].execute()
          proc.waitForProcessOutput(System.out, System.err)
        }
      }
    }
  }
}