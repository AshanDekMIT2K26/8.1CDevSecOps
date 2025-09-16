pipeline {
  agent any

  environment {
    // Adjust if your Node is elsewhere. These are the usual defaults.
    NODE_DIR = 'C:\\Program Files\\nodejs'
    NPM_ROAMING = 'C:\\Users\\Ashan indika\\AppData\\Roaming\\npm'
  }

  stages {
    stage('Check Tools') {
      steps {
        withEnv(["PATH=${NODE_DIR};${NPM_ROAMING};%PATH%"]) {
          bat 'where node || echo node not found'
          bat 'where npm || echo npm not found'
          bat 'node -v || exit /b 1'
          bat 'npm -v || exit /b 1'
        }
      }
    }

    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/AshanDekMIT2K26/8.1CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        withEnv(["PATH=${NODE_DIR};${NPM_ROAMING};%PATH%"]) {
          bat 'npm install'
        }
      }
    }

    stage('Run Tests') {
      steps {
        withEnv(["PATH=${NODE_DIR};${NPM_ROAMING};%PATH%"]) {
          bat 'cmd /c npm test || exit /b 0'
        }
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: 'reports\\junit\\*.xml'
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        withEnv(["PATH=${NODE_DIR};${NPM_ROAMING};%PATH%"]) {
          bat 'cmd /c npm run coverage || exit /b 0'
          archiveArtifacts artifacts: 'coverage\\**\\*', allowEmptyArchive: true
        }
      }
    }

    stage('NPM Audit') {
      steps {
        withEnv(["PATH=${NODE_DIR};${NPM_ROAMING};%PATH%"]) {
          bat 'cmd /c npm audit --json > npm-audit.json || exit /b 0'
          bat 'cmd /c npm audit || exit /b 0'
        }
      }
      post {
        always {
          archiveArtifacts artifacts: 'npm-audit.json, npm-debug.log, **\\npm-audit*.json', allowEmptyArchive: true
        }
      }
    }
  }

  post {
    success { echo '✅ DevSecOps basics pipeline finished.' }
    always  { echo "Result: ${currentBuild.currentResult}" }
  }
}
