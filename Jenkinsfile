import groovy.json.JsonOutput

pipeline {
    agent any

    environment {
        JENKINS_URL = "http://localhost:8080"
        API_URL = "https://2cf9-2402-e280-3e1d-bce-2584-894f-4e39-6c7c.ngrok-free.app/jenkins-metadata"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                script {
                    echo "Checking out from source control..."
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
                    def endTime = System.currentTimeMillis()
                    echo "Repository cloned successfully in ${endTime - startTime} ms"
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    echo 'Building the project...'
                    def endTime = System.currentTimeMillis()
                    echo "Build completed in ${endTime - startTime} ms"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    echo 'Running tests...'
                    def endTime = System.currentTimeMillis()
                    echo "Tests executed successfully in ${endTime - startTime} ms"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    echo 'Deploying the application...'
                    def endTime = System.currentTimeMillis()
                    echo "Deployment completed successfully in ${endTime - startTime} ms"
                }
            }
        }

        stage('Fetch Real-Time Data') {
            steps {
                script {
                    def jobName = env.JOB_NAME ?: "unknown-job"
                    def buildNumber = env.BUILD_NUMBER ?: "unknown-build"

                    def response = sh(script: """curl -s -X GET "$JENKINS_URL/job/$jobName/$buildNumber/wfapi/describe" """, returnStdout: true).trim()
                    echo "Real-time Pipeline Data: ${response}"

                    sh """curl -X POST "$API_URL" \
                        -H "Content-Type: application/json" \
                        -d '${response}'"""
                }
            }
        }
    }
}
