pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/sanvi-verma/CountdownTimer.git'
            }
        }

        stage('Set Up Environment') {
            steps {
                echo 'Setting up environment...'
                // Add any necessary setup commands here (e.g., install dependencies if needed)
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
                // Add build steps (if applicable)
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                // Add test steps (if applicable)
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
