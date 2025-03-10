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
                    npm install -g eslint
                    if [ -f eslint.config.js ]; then
                        node --experimental-modules --loader ts-node/esm eslint.config.js || true
                    fi
                    npx eslint . --fix || true
                '''
            }
        }

        stage('Build Project') {
            steps {
                sh '''
                    if npm run | grep -q "build"; then
                        npm run build
                    else
                        echo "Skipping build: No build script defined"
                    fi
                '''
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
                    if npm run | grep -q "deploy"; then
                        npm run deploy
                    else
                        echo "Skipping deployment: No deploy script defined"
                    fi
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
