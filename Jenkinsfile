pipeline {
  agent any

  options { timestamps() }

  environment {
    PATH = "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:${env.PATH}"

    SONAR_HOST_URL = "https://sonarcloud.io"
    SONAR_ORG      = "anthonybnc"
    SONAR_PROJECT  = "AnthonyBnc_8.2CDevSecOps"

    // change this to the email you want to receive notifications
    DEV_EMAIL = "nguyenchungbaoan@gmail.com"
  }

  stages {

    stage('Checkout (Tool: Git SCM)') {
      steps { checkout scm }
    }

    stage('Verify Node & NPM (Tool: Node.js)') {
      steps {
        sh '''
          which node || true
          which npm  || true
          node -v
          npm -v
        '''
      }
    }

    stage('Install Dependencies (Tool: npm)') {
      steps { sh 'npm install' }
    }

    stage('Run Tests (Tool: Snyk)') {
      steps {
        script {
          // capture logs into a file so we can attach it
          def rc = sh(script: 'npm test > test.log 2>&1', returnStatus: true)

          // send email regardless of success/failure
          emailext(
            to: env.DEV_EMAIL,
            subject: "[Jenkins] TEST stage - ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${rc == 0 ? 'SUCCESS' : 'FAILURE'}",
            body: """Test stage finished.

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Status: ${rc == 0 ? 'SUCCESS' : 'FAILURE'}
Console: ${env.BUILD_URL}console

Log file attached: test.log
""",
            attachmentsPattern: "test.log"
          )

          // keep pipeline running (assignment wants email even on failure)
          if (rc != 0) {
            echo "Tests failed (non-blocking for assignment)."
          }
        }
      }
    }

    stage('SonarCloud Analysis (Tool: SonarCloud + SonarScanner)') {
      steps {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
          sh '''
            echo "Running SonarCloud analysis..."
            if [ ! -d sonar-scanner-5.0.1.3006-macosx ]; then
              echo "Downloading SonarScanner..."
              curl -sSLo sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-macosx.zip
              unzip -o sonar-scanner.zip
            fi

            ./sonar-scanner-5.0.1.3006-macosx/bin/sonar-scanner \
              -Dsonar.host.url=${SONAR_HOST_URL} \
              -Dsonar.organization=${SONAR_ORG} \
              -Dsonar.projectKey=${SONAR_PROJECT} \
              -Dsonar.sources=. \
              -Dsonar.exclusions=node_modules/**,test/** \
              -Dsonar.sourceEncoding=UTF-8 \
              -Dsonar.token=$SONAR_TOKEN
          '''
        }
      }
    }

    stage('NPM Audit (Security Scan Tool: npm audit)') {
      steps {
        script {
          def rc = sh(script: 'npm audit > audit.log 2>&1', returnStatus: true)

          emailext(
            to: env.DEV_EMAIL,
            subject: "[Jenkins] SECURITY SCAN stage - ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${rc == 0 ? 'SUCCESS' : 'FAILURE'}",
            body: """Security scan finished.

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Status: ${rc == 0 ? 'SUCCESS' : 'FAILURE'}
Console: ${env.BUILD_URL}console

Log file attached: audit.log
""",
            attachmentsPattern: "audit.log"
          )

          // keep pipeline running for assignment evidence
          if (rc != 0) {
            echo "npm audit found vulnerabilities (non-blocking for assignment)."
          }
        }
      }
    }
  }

  post {
    always {
      echo "Pipeline finished (results recorded)."
    }
  }
}
