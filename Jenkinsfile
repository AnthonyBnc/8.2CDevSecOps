pipeline {
  agent any

  environment {
    PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:${env.PATH}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Install Dependencies') {
      steps {
        sh 'which node || true'
        sh 'which npm || true'
        sh 'node -v'
        sh 'npm -v'
        sh 'npm install'
      }
    }

    stage('Run Tests') {
      steps { sh 'npm test || true' }
    }

    stage('Generate Coverage Report') {
      steps { sh 'npm run coverage || true' }
    }

    stage('SonarCloud Analysis') {
  steps {
    withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
      sh '''
        echo "Downloading SonarScanner..."
        curl -sSLo sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-macosx.zip
        unzip -o sonar-scanner.zip

        echo "Running SonarCloud analysis..."
        ./sonar-scanner-*/bin/sonar-scanner \
          -Dsonar.token=$SONAR_TOKEN
      '''
    }
  }
}

    stage('NPM Audit (Security Scan)') {
      steps { sh 'npm audit || true' }
    }
  }
}
