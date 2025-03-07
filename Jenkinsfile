pipeline {
    agent any

    environment {
        PATH = "/var/jenkins_home/.npm-global/bin:$PATH"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/sanvi-verma/CountdownTimer.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    mkdir -p /var/jenkins_home/.npm-global
                    npm config set prefix '/var/jenkins_home/.npm-global'
                    npm install
                '''
            }
        }

        stage('Lint JavaScript') {
            steps {
                sh '''
                    npm install -g eslint
                    eslint .
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
                sh '''
                    git config --global user.email "your-email@example.com"
                    git config --global user.name "Sanvi Verma"
                    npm run deploy
                '''
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
