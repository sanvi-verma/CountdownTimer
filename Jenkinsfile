import groovy.json.JsonOutput
import java.util.Base64

pipeline {
    agent any

    environment {
        JENKINS_URL = "http://localhost:8080"
        API_URL = "https://2cf9-2402-e280-3e1d-bce-2584-894f-4e39-6c7c.ngrok-free.app/jenkins-metadata"
         API_KEY = 'qteyew2537e3ygdhusdhd833' 

    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying the application...'
            }
        }

        stage('Send Metadata') {
            steps {
                script {
                    def jobName = env.JOB_NAME ?: "unknown-job"
                    def buildNumber = env.BUILD_NUMBER ?: "unknown-build"

                    sh """
                        curl -X POST "$API_URL" \
                        -H "Content-Type: application/json" \
                        -d "\$(curl -s -X GET "$JENKINS_URL/job/$jobName/$buildNumber/wfapi/describe")"
                    """
                }
            }
        }
    }
}
