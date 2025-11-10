pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/AnusriMurugan20/JenkiRepo.git'
            }
        }

        stage('Show workspace (debug)') {
            steps {
                bat 'echo === workspace listing === & dir /S /B'
            }
        }

        stage('Install, Build & Test') {
            steps {
                // use PowerShell from a single bat step so it runs on Windows nodes
                bat '''
powershell -NoProfile -Command ^
$p = Get-ChildItem -Recurse -Filter package.json -File -ErrorAction SilentlyContinue | Select-Object -First 1; ^
if ($null -eq $p) { Write-Host "ERROR: package.json not found in workspace"; exit 1 } ^
$dir = $p.DirectoryName; ^
Write-Host "Found package.json in: $dir"; ^
Set-Location $dir; ^
Write-Host "Node version:"; node -v; Write-Host "Npm version:"; npm -v; ^
Write-Host "Running npm install..."; npm install; ^
Write-Host "Running npm run build..."; npm run build; ^
Write-Host "Running tests..."; npm test -- --watchAll=false --passWithNoTests
'''
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
