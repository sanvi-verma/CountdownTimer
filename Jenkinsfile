import groovy.json.JsonOutpu
pipeline {
    agent any
      environment {
        JENKINS_URL = "http://localhost:8080"
        API_URL = "https://2cf9-2402-e280-3e1d-bce-2584-894f-4e39-6c7c.ngrok-free.app/jenkins-metadata"
        API_KEY = "qteyew2537e3ygdhusdhd833"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'

            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t my-static-site .'
            }
        }

        stage('Run Container') {
            steps {
                echo 'Running Docker container on port 8000...'
                sh 'docker stop my-static-site-container || true'
                sh 'docker rm my-static-site-container || true'
                sh 'docker run -d --name my-static-site-container -p 8000:80 my-static-site'
            }
        }
        stage('Print Environment Variables') {
            steps {
                script {
                    sh 'env' // Lists all environment varia
                }
            }
        }
    }
    post {
    always {
        script {
            

                  def jobName = env.JOB_NAME ?: "unknown-job"
                    def buildNumber = env.BUILD_NUMBER ?: "unknown-build"

                    def response = sh(script: """curl -s -X GET "$JENKINS_URL/job/$jobName/$buildNumber/wfapi/describe" """, returnStdout: true).trim()
                    echo "Real-time Pipeline Data: ${response}"

                    sh """curl -X POST "$API_URL" \
                        -H "Content-Type: application/json" \
                        -H "x-api-key: $API_KEY" \
                        -d '${response}'"""

        }
    }
}



}
