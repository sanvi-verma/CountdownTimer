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
                        sendMetadata("Clone Repository", "SUCCESS", startTime, endTime)
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        sendMetadata("Clone Repository", "FAILURE", startTime, endTime)
                        error("Stage failed: ${e}")
                    }
                }
            }
        }

        stage('Set Up Environment') {
            steps {
                script {
                    def startTime = new Date().getTime()
                    try {
                        echo 'Setting up environment...'
                        def endTime = new Date().getTime()
                        sendMetadata("Set Up Environment", "SUCCESS", startTime, endTime)
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        sendMetadata("Set Up Environment", "FAILURE", startTime, endTime)
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
                        sendMetadata("Build", "SUCCESS", startTime, endTime)
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        sendMetadata("Build", "FAILURE", startTime, endTime)
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
                        sendMetadata("Test", "SUCCESS", startTime, endTime)
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        sendMetadata("Test", "FAILURE", startTime, endTime)
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
                        sendMetadata("Deploy", "SUCCESS", startTime, endTime)
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        sendMetadata("Deploy", "FAILURE", startTime, endTime)
                        error("Deployment failed due to missing config.")
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

def sendMetadata(stageName, status, startTime, endTime) {
    def duration = endTime - startTime
    def consoleLog = currentBuild.rawBuild.getLog(100).join("\n").replaceAll("\"", "'")

    def stepData = [
        "stage"       : stageName,
        "status"      : status,
        "startTime"   : startTime.toString(),
        "endTime"     : endTime.toString(),
        "duration"    : duration.toString(),
        "consoleLog"  : consoleLog
    ]

    def json = new groovy.json.JsonBuilder(stepData).toString()

    sh """curl -X POST ${API_URL} \
        -H "Content-Type: application/json" \
        -d '${json}'"""
}

def sendFinalMetadata(pipelineStatus) {
    def metadata = [
        "metadata": [
            "buildNumber"    : env.BUILD_NUMBER,
            "jobName"        : env.JOB_NAME,
            "nodeName"       : env.NODE_NAME,
            "executorNumber" : env.EXECUTOR_NUMBER ?: "N/A",
            "buildUrl"       : env.BUILD_URL
        ],
        "status": pipelineStatus
    ]

    def json = new groovy.json.JsonBuilder(metadata).toString()

    sh """curl -X POST ${API_URL} \
        -H "Content-Type: application/json" \
        -d '${json}'"""
}
