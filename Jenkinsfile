// Jenkinsfile (Declarative) - Windows agent - auto-detects Node / Maven / .NET
pipeline {
  agent { label 'windows' }                 // change label to your Windows agent label
  options {
    timeout(time: 60, unit: 'MINUTES')     // overall timeout
    buildDiscarder(logRotator(numToKeepStr: '20')) // keep last 20 builds
  }
  environment {
    // Example of credentials usage (set credential in Jenkins and use its id here)
    // GIT_CRED = credentials('git-creds-id')
    ARTIFACT_DIR = "artifacts"
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Detect & Setup') {
      steps {
        script {
          isNode = fileExists('package.json')
          isMaven = fileExists('pom.xml')
          isDotnet = false
          // crude detection for .NET: any *.csproj in repo root or subfolders
          dir('.') {
            isDotnet = sh(returnStatus: true, script: 'bash -c "ls **/*.csproj 2>/dev/null || true"') == 0 ? true : false
          }
          // Because we're on Windows use a fallback detection with Groovy:
          if (!isDotnet) {
            isDotnet = findFiles(glob: '**/*.csproj').length > 0
          }
          echo "Project types detected => Node: ${isNode}, Maven: ${isMaven}, .NET: ${isDotnet}"
        }
      }
    }

    stage('Install Dependencies') {
      steps {
        script {
          if (isNode) {
            bat 'npm ci'
          } else if (isMaven) {
            bat 'mvn -B -DskipTests=true verify'
          } else if (isDotnet) {
            bat 'dotnet restore'
          } else {
            echo 'No recognized project files (package.json, pom.xml, or .csproj). Skipping dependency step.'
          }
        }
      }
    }

    stage('Build') {
      steps {
        script {
          if (isNode) {
            bat 'npm run build || echo "No build script defined"'
          } else if (isMaven) {
            bat 'mvn -B -DskipTests=true package'
          } else if (isDotnet) {
            bat 'dotnet build --configuration Release'
          } else {
            echo 'No build step performed.'
          }
        }
      }
    }

    stage('Test') {
      steps {
        script {
          if (isNode) {
            // If your tests produce junit xml, adapt the path
            bat 'npm test || set TEST_ERROR=%ERRORLEVEL% & exit /b %TEST_ERROR%'
          } else if (isMaven) {
            bat 'mvn -B test'
            junit '**/target/surefire-reports/*.xml'
          } else if (isDotnet) {
            // dotnet test prints results; to publish JUnit-like results you may add a logger to produce TRX and convert
            bat 'dotnet test --logger "trx;LogFileName=test_results.trx"'
            // Archive the TRX so you can download if conversion not configured
            archiveArtifacts artifacts: '**/TestResults/*.trx', allowEmptyArchive: true
          } else {
            echo 'No tests run.'
          }
        }
      }
    }

    stage('Archive Artifacts') {
      steps {
        script {
          bat "mkdir ${env.ARTIFACT_DIR} 2>nul || exit 0"
          if (isNode) {
            bat "if exist dist ( xcopy /E /I dist ${env.ARTIFACT_DIR}\\dist )"
            archiveArtifacts artifacts: "${env.ARTIFACT_DIR}/**/*", allowEmptyArchive: true
          } else if (isMaven) {
            archiveArtifacts artifacts: 'target/*.jar, target/*.war', fingerprint: true, allowEmptyArchive: true
          } else if (isDotnet) {
            // publish the publish output if exists
            bat 'dotnet publish -c Release -o publish || exit 0'
            archiveArtifacts artifacts: 'publish/**', fingerprint: true, allowEmptyArchive: true
          } else {
            archiveArtifacts artifacts: '**/*', allowEmptyArchive: true
          }
        }
      }
    }
  }

  post {
    success {
      echo "Build succeeded: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
    }
    unstable {
      echo "Build unstable."
    }
    failure {
      echo "Build failed. See console output."
    }
    always {
      // cleanup workspace if you want:
      cleanWs()
    }
  }
}
