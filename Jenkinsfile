pipeline {
    agent any

    environment {
        PATH = "/var/jenkins_home/.npm-global/bin:$PATH"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    mkdir -p /var/jenkins_home/.npm-global
                    npm config set prefix '/var/jenkins_home/.npm-global'
                    npm install --legacy-peer-deps
                '''
            }
        }

        stage('Lint JavaScript') {
            steps {
                sh '''
                    npx eslint . --fix || true
                '''
            }
        }

        stage('Build Project') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Deploy to GitHub Pages') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config --global user.email "your-email@example.com"
                        git config --global user.name "Sanvi Verma"
                        git remote set-url origin https://sanvi-verma:${GITHUB_TOKEN}@github.com/sanvi-verma/CountdownTimer.git
                        GIT_ASKPASS=echo npm run deploy
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
