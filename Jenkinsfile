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
    SONAR_HOST_URL           = 'https://sonarcloud.io'
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

    // --- SonarCloud Analysis (securely uses SONAR_TOKEN) ---
    stage('SonarCloud Analysis') {
      steps {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
          script {
            // CLI version
            def ver = '5.0.1.3006'
            def zipUrl = "https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-${ver}-windows.zip"
            def work = env.WORKSPACE.replace('/', '\\')
            def dlDir = "${work}\\sonarcli"
            def zipPath = "${dlDir}\\sonar-scanner.zip"

            // 1) Prepare folder + download zip (PowerShell for reliable TLS/UNZIP on Windows)
            powershell """
              New-Item -ItemType Directory -Force -Path '${dlDir}' | Out-Null
              Write-Host 'Downloading SonarScanner from ${zipUrl}'
              Invoke-WebRequest -Uri '${zipUrl}' -OutFile '${zipPath}'
              # 2) Extract
              Expand-Archive -LiteralPath '${zipPath}' -DestinationPath '${dlDir}' -Force
            """

            // 3) Find the extracted scanner dir and run it
            // The extracted folder is like sonar-scanner-${ver}-windows\\bin\\sonar-scanner.bat
            def scannerBat = "sonar-scanner-${ver}-windows\\bin\\sonar-scanner.bat"
            def scannerPath = "${dlDir}\\${scannerBat}"

            // Optional JVM opts (increase memory if needed)
            def sonarScannerOpts = "-Xmx512m"

            // 4) Execute scan; relies on sonar-project.properties in repo.
            //    Console will show HTTP 200 responses from SonarCloud if OK.
            bat """
              set SONAR_SCANNER_OPTS=${sonarScannerOpts}
              \"${scannerPath}\" -Dsonar.host.url=%SONAR_HOST_URL% -Dsonar.login=%SONAR_TOKEN%
            """
          }
        }
      }
    }
  }


  post {
    success { echo '✅ DevSecOps basics pipeline finished.' }
    always  { echo "Result: ${currentBuild.currentResult}" }
  }
}
