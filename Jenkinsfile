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
                    def pipelineJson = readJSON text: pipelineData

                    // Fetch Build Details
                    def buildData = sh(script: """curl -s -X GET "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/api/json?depth=1" """, returnStdout: true).trim()
                    def buildJson = readJSON text: buildData

                    // Fetch Git Commit and Branch from Environment Variables
                    def gitBranch = sh(script: 'echo $GIT_BRANCH', returnStdout: true).trim()
                    def gitCommit = sh(script: 'echo $GIT_COMMIT', returnStdout: true).trim()

                    // Construct JSON payload
                    def payload = [
                        pipelineData: pipelineJson,
                        buildData: buildJson,
                        git: [
                            branch: gitBranch,
                            commit: gitCommit
                        ]
                    ]

                    // Convert to JSON string
                    def payloadJson = writeJSON returnText: true, json: payload

                    echo "Final JSON Payload: ${payloadJson}"

                    // Send Data to API
                    sh """curl -v -X POST "$API_URL" \
                        -H "Content-Type: application/json" \
                        -d '${payloadJson}' """
                }
            }
        }
    }
}
