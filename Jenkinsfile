pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
            }
        }

        stage('Lint JavaScript') {
            steps {
                echo 'Checking JavaScript code quality...'
                sh 'npm install eslint -g'
                sh 'eslint . || true' // Ignore errors but still show output
            }
        }

        stage('Deploy to GitHub Pages') {
            steps {
                echo 'Deploying to GitHub Pages...'
                sh '''
                git config --global user.email "your-email@example.com"
                git config --global user.name "Sanvi Verma"
                git checkout --orphan gh-pages
                git rm -rf .
                git checkout main -- .
                git commit -m "Deploy to GitHub Pages"
                git push --force origin gh-pages
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
