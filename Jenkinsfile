pipeline {
    agent any
    tools { nodejs 'nodejs' }
    stages {
        stage('Checkout') {
            steps { git branch: 'main', url: 'https://github.com/AnusriMurugan20/JenkiRepo.git' }
        }
        stage('Sanity / list files') {
            steps { bat 'dir /S /B' }
        }
        stage('Install Dependencies') {
            steps {
                dir('frontend') {
                    bat 'if exist package.json (echo package.json FOUND) else (echo package.json MISSING & exit /b 1)'
                    bat 'npm install'
                }
            }
        }
        stage('Build') {
            steps {
                dir('frontend') { bat 'npm run build' }
            }
        }
        stage('Test') {
            steps {
                dir('frontend') { bat 'npm test -- --watchAll=false --passWithNoTests' }
            }
        }
    }
}
