pipeline {
  agent any

  options {
    timestamps()
  }

  environment {
    // Ensure Jenkins can find Node/NPM on macOS
    PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:${env.PATH}"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Verify Node & NPM') {
      steps {
        sh '''
          which node || true
          which npm  || true
          node -v
          npm -v
        '''
      }
    }

    stage('Install Dependencies') {
      steps {
        sh 'npm install'
      }
    }

    stage('Run Tests (non-blocking)') {
      steps {
        sh '''
          # nodejs-goof runs snyk test, which requires auth.
          # Allow pipeline to continue for assessment purposes.
          npm test || true
        '''
      }
    }

    stage('Generate Coverage Report (non-blocking)') {
      steps {
        sh '''
          # Some repos do not define a coverage script.
          npm run coverage || true
        '''
      }
    }

    stage('SonarCloud Analysis') {
      steps {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
          sh '''
            echo "Running SonarCloud analysis..."

            if [ ! -d "sonar-scanner-5.0.1.3006-macosx" ]; then
              curl -sSLo sonar-scanner.zip \
                https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-macosx.zip
              unzip -o sonar-scanner.zip
            fi

            ./sonar-scanner-5.0.1.3006-macosx/bin/sonar-scanner \
              -Dsonar.host.url=https://sonarcloud.io \
              -Dsonar.organization=anthonybnc \
              -Dsonar.projectKey=AnthonyBnc_8.2CDevSecOps \
              -Dsonar.sources=. \
              -Dsonar.exclusions=node_modules/**,test/** \
              -Dsonar.sourceEncoding=UTF-8 \
              -Dsonar.token=$SONAR_TOKEN
          '''
        }
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
        sh '''
          # Display vulnerabilities without failing the build
          npm audit || true
        '''
      }
    }
  }

  post {
    always {
      echo "Pipeline finished (results recorded)."
    }
  }
}
