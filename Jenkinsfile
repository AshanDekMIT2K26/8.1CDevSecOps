pipeline {
  agent any

  tools {
    nodejs 'Node18' 
  }

  environment {
    DIRECTORY_PATH          = '.'
    TESTING_ENVIRONMENT     = 'staging'
    PRODUCTION_ENVIRONMENT  = 'production'
    // Optional: lock npm cache dir so Jenkins can reuse it between builds
    NPM_CONFIG_CACHE        = "${WORKSPACE}\\.npm-cache"
  }

  stages {
    // stage('Check Tools') {
    //   steps {
    //     bat 'where node || echo node not found'
    //     bat 'where npm  || echo npm not found'
    //     bat 'node -v'
    //     bat 'npm -v'
    //   }
    // }

    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/AshanDekMIT2K26/8.1CDevSecOps.git'
      }
    }

    stage('Install Dependencies') {
      steps {
        // Faster + reproducible if lockfile exists
        bat 'if exist package-lock.json ( npm ci ) else ( npm install )'
      }
    }

    stage('Run Tests') {
      steps {
        // Don’t fail pipeline if tests fail (per task brief)
        bat 'cmd /c npm test || exit /b 0'
      }
      post {
        always {
          junit allowEmptyResults: true, testResults: 'reports\\junit\\*.xml'
        }
      }
    }

    stage('Generate Coverage Report') {
      steps {
        bat 'cmd /c npm run coverage || exit /b 0'
      }
      post {
        always {
          archiveArtifacts artifacts: 'coverage\\**\\*', allowEmptyArchive: true
        }
      }
    }

    stage('NPM Audit (Security Scan)') {
      steps {
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
