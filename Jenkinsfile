pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                 git branch: 'main',url: 'https://github.com/sanvi-verma/CountdownTimer.git'  // Replace with your repo URL
            }
        }

        stage('Lint Code') {
            steps {
                script {
                    echo 'Linting HTML, CSS, and JavaScript files...'
                    sh 'npm install -g htmlhint csslint eslint'  // Install linters if not installed
                    sh 'htmlhint *.html || true'
                    sh 'csslint *.css || true'
                    sh 'eslint *.js --fix || true'  // Fix minor issues automatically
                }
            }
        }

        stage('Commit and Push Changes') {
            steps {
                script {
                    sh '''
                    git config --global user.email "your-email@example.com"
                    git config --global user.name "Jenkins"
                    git add .
                    git commit -m "Auto-update by Jenkins"
                    git push origin main  # Change branch if needed
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment to GitHub Pages successful!'
        }
        failure {
            echo 'Pipeline failed. Check the logs.'
        }
    }
}
