pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/sanvi-verma/CountdownTimer.git'
        BRANCH = 'main'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Lint Code') {
    steps {
        script {
            echo 'Linting HTML, CSS, and JavaScript files...'
            sh 'npm install htmlhint csslint eslint'  // Install locally in node_modules
            sh 'npx htmlhint index.html || true'     // Run linters using npx
            sh 'npx csslint style.css || true'
            sh 'npx eslint eslint.config.js index.js --fix || true'
        }
    }
}

        stage('Commit and Push Changes') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-token', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    script {
                        sh '''
                        git config --global user.email "your-email@example.com"
                        git config --global user.name "Jenkins"
                        git add .
                        git commit -m "Auto-update by Jenkins" || true
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/sanvi-verma/CountdownTimer.git main
                        '''
                    }
                }
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Check the logs.'
        }
    }
}
