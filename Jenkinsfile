pipeline {
    agent any

    environment {
        JENKINS_URL = "http://localhost:8080"
        API_URL = "https://2cf9-2402-e280-3e1d-bce-2584-894f-4e39-6c7c.ngrok-free.app/jenkins-metadata"
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo 'Building the project...'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo 'Running tests...'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying the application...'
                }
            }
        }

        stage('Fetch Real-Time Data') {
            steps {
                script {
                    // Fetch Pipeline Metadata
                    def pipelineData = sh(script: """curl -s -X GET "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/wfapi/describe" """, returnStdout: true).trim()

                    // Fetch Build Details
                    def buildData = sh(script: """curl -s -X GET "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/api/json?depth=1" """, returnStdout: true).trim()

                    // Fetch Git Data
                    def gitBranch = sh(script: 'echo $GIT_BRANCH', returnStdout: true).trim()
                    def gitCommit = sh(script: 'echo $GIT_COMMIT', returnStdout: true).trim()
                    def gitData = """{ "branch": "${gitBranch}", "commit": "${gitCommit}" }"""

                    // Create Payload
                    def payload = """{
                        "pipeline": ${pipelineData},
                        "build": ${buildData},
                        "git": ${gitData}
                    }"""

                    echo "Sending Data: ${payload}"

                    // Send Data to API
                    sh """curl -X POST "$API_URL" \
                        -H "Content-Type: application/json" \
                        -d '${payload}'"""
                }
            }
        }
    }
}
