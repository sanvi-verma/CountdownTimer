pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/sanvi-verma/CountdownTimer.git'
            }
        }
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                git config --global user.email "your-email@example.com"
                git config --global user.name "Sanvi Verma"
                git remote set-url origin https://sanvi-verma:${GITHUB_TOKEN}@github.com/sanvi-verma/CountdownTimer.git
                git add .
                git commit -m "Deploying to GitHub Pages"
                git push origin main
                npm run deploy
                '''
            }
        }
    }
}
