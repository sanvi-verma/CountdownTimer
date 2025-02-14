pipeline {
    agent any
    
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
            }
        }
        
        stage('Set Up Environment') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'echo "Setting up environment..."'
                    } else {
                        bat 'echo Setting up environment...'
                    }
                }
            }
        }

        stage('Install Dependencies') {
    steps {
        dir('CountdownTimer') { // Change this if your repo has a subfolder
            sh 'npm install'
        }
    }
}


        stage('Build') {
            steps {
                sh 'npm run build' // Adjust based on your project setup
            }
        }

        stage('Test') {
            steps {
                sh 'npm test' // Runs tests if applicable
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying the application...'
                // Add deployment steps here
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
