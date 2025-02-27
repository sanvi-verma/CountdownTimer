pipeline {
    agent any

    options {
        timestamps()  // Adds timestamps in logs
        buildDiscarder(logRotator(numToKeepStr: '10'))  // Keep last 10 builds
    }

    environment {
        API_URL = 'https://4ad2-2402-e280-3e1d-bce-6df3-1e62-d8d0-f624.ngrok-free.app/jenkins-metadata'
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    def startTime = new Date().getTime()
                    try {
                        git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
                        def endTime = new Date().getTime()
                        sendMetadata("Clone Repository", "SUCCESS", startTime, endTime, "Cloning into repo...")
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        sendMetadata("Clone Repository", "FAILURE", startTime, endTime, e.toString())
                        error("Stage failed: ${e}")
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    def startTime = new Date().getTime()
                    try {
                        echo 'Building the project...'
                        def endTime = new Date().getTime()
                        sendMetadata("Build", "SUCCESS", startTime, endTime, "Compiling source code...")
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        sendMetadata("Build", "FAILURE", startTime, endTime, e.toString())
                        error("Stage failed: ${e}")
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    def startTime = new Date().getTime()
                    try {
                        echo 'Running tests...'
                        def endTime = new Date().getTime()
                        sendMetadata("Test", "SUCCESS", startTime, endTime, "Running tests...")
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        sendMetadata("Test", "FAILURE", startTime, endTime, e.toString())
                        error("Stage failed: ${e}")
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def startTime = new Date().getTime()
                    try {
                        echo 'Deploying the application...'
                        def endTime = new Date().getTime()
                        sendMetadata("Deploy", "SUCCESS", startTime, endTime, "Deployment successful.")
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        sendMetadata("Deploy", "FAILED", startTime, endTime, "Deployment failed due to missing config.")
                        error("Stage failed: ${e}")
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                sendFinalMetadata("SUCCESS")
                echo 'Pipeline executed successfully!'
            }
        }
        failure {
            script {
                sendFinalMetadata("FAILURE")
                echo 'Pipeline failed!'
            }
        }
    }
}

def stepsData = []

def sendMetadata(stageName, status, startTime, endTime, consoleLog) {
    def duration = endTime - startTime

    stepsData << [
        stage      : stageName,
        status     : status,
        startTime  : startTime.toString(),
        endTime    : endTime.toString(),
        duration   : duration.toString(),
        consoleLog : consoleLog
    ]
}

def sendFinalMetadata(status) {
    def metadata = [
        metadata: [
            buildNumber    : env.BUILD_NUMBER,
            jobName        : env.JOB_NAME,
            nodeName       : env.NODE_NAME,
            executorNumber : env.EXECUTOR_NUMBER ?: "0",
            buildUrl       : env.BUILD_URL
        ],
        steps: stepsData
    ]

    def json = groovy.json.JsonOutput.toJson(metadata)

    sh """curl -X POST ${API_URL} \
        -H "Content-Type: application/json" \
        -d '${json}'"""
}
