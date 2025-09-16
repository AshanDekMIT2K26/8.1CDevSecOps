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
        sh 'npm install'
      }
    }

    stage('Run Tests') {
      steps {
        // Continue even if tests fail, as per brief
        sh 'npm test || true'
      }
      post {
        always {
          // If you export JUnit reports, publish them (optional)
          junit allowEmptyResults: true, testResults: 'reports/junit/*.xml'
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        // Ensure package.json has a "coverage" script, e.g. "nyc --reporter=lcov npm test"
        sh 'npm run coverage || true'
        archiveArtifacts artifacts: 'coverage/**/*', allowEmptyArchive: true
      }
    }

    stage('NPM Audit') {
      steps {
        // Generate a machine-readable report for artifacts
        sh 'npm audit --json || true'
        sh 'npm audit || true'
      }
      post {
        always {
          archiveArtifacts artifacts: 'npm-*audit*.json, npm-debug.log, **/npm-audit*.json', allowEmptyArchive: true
        }
      }
    }
  }

  post {
    success { echo '✅ DevSecOps basics pipeline finished.' }
    always  { echo "Result: ${currentBuild.currentResult}" }
  }
}
