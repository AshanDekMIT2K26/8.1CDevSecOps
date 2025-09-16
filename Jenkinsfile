pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/AshanDekMIT2K26/8.1CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        bat 'npm install'
      }
    }

    stage('Run Tests') {
      steps {
        // Continue even if tests fail, as per brief
        bat 'cmd /c npm test || exit /b 0'
      }
      post {
        always {
          // If you export JUnit reports, publish them (optional)
          junit allowEmptyResults: true, testResults: 'reports\\junit\\*.xml'
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        // Ensure package.json has a "coverage" script, e.g. "nyc --reporter=lcov npm test"
        bat 'cmd /c npm run coverage || exit /b 0'
        archiveArtifacts artifacts: 'coverage\\**\\*', allowEmptyArchive: true
      }
    }

    stage('NPM Audit') {
      steps {
        // Generate a machine-readable report for artifacts
        bat 'cmd /c npm audit --json > npm-audit.json || exit /b 0'
        bat 'cmd /c npm audit || exit /b 0'
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
