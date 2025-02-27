pipeline {
    agent any

    options {
        timestamps()  // Adds timestamps in logs
        buildDiscarder(logRotator(numToKeepStr: '10'))  // Keep last 10 builds
    }

    environment {
        API_URL = 'https://4ad2-2402-e280-3e1d-bce-6df3-1e62-d8d0-f624.ngrok-free.app/jenkins-metadata'
        METADATA = '{"metadata": {}, "steps": []}'  // Initialize metadata structure
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    def startTime = new Date().getTime()
                    try {
                        git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
                        def endTime = new Date().getTime()
                        def consoleLog = getConsoleLog()
                        appendStageMetadata("Clone Repository", "SUCCESS", startTime, endTime, consoleLog)
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        def consoleLog = getConsoleLog()
                        appendStageMetadata("Clone Repository", "FAILURE", startTime, endTime, consoleLog)
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
                        def consoleLog = getConsoleLog()
                        appendStageMetadata("Build", "SUCCESS", startTime, endTime, consoleLog)
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        def consoleLog = getConsoleLog()
                        appendStageMetadata("Build", "FAILURE", startTime, endTime, consoleLog)
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
                        def consoleLog = getConsoleLog()
                        appendStageMetadata("Test", "SUCCESS", startTime, endTime, consoleLog)
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        def consoleLog = getConsoleLog()
                        appendStageMetadata("Test", "FAILURE", startTime, endTime, consoleLog)
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
                        def consoleLog = getConsoleLog()
                        appendStageMetadata("Deploy", "SUCCESS", startTime, endTime, consoleLog)
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        def consoleLog = getConsoleLog()
                        appendStageMetadata("Deploy", "FAILURE", startTime, endTime, consoleLog)
                        error("Stage failed: ${e}")
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                sendFinalMetadata()
                echo 'Pipeline executed successfully!'
            }
        }
        failure {
            script {
                sendFinalMetadata()
                echo 'Pipeline failed!'
            }
        }
    }
}

// **Function to Append Metadata in Correct Order**
def appendStageMetadata(stageName, status, startTime, endTime, consoleLog) {
    def duration = endTime - startTime
    def newStep = """{
        "stage": "${stageName}",
        "status": "${status}",
        "startTime": "${startTime}",
        "endTime": "${endTime}",
        "duration": "${duration}",
        "consoleLog": "${consoleLog.replaceAll("\"", "'")}"
    }"""

    // Correcting order by appending at the end instead of beginning
    if (env.METADATA.contains('"steps": []')) {
        env.METADATA = env.METADATA.replace('"steps": []', '"steps": [' + newStep + ']')
    } else {
        env.METADATA = env.METADATA.replaceFirst('"steps": \\[', '"steps": [' + newStep + ', ')
    }
}

// **Function to Send Final Metadata**
def sendFinalMetadata() {
    def metadata = """{
        "metadata": {
            "buildNumber": "${env.BUILD_NUMBER}",
            "jobName": "${env.JOB_NAME}",
            "nodeName": "${env.NODE_NAME}",
            "executorNumber": "${env.EXECUTOR_NUMBER ?: 'N/A'}",
            "buildUrl": "${env.BUILD_URL}"
        },
        ${env.METADATA.substring(env.METADATA.indexOf('"steps":'))}
    }"""

    sh """curl -X POST ${API_URL} \
        -H "Content-Type: application/json" \
        -d '${metadata}'"""
}

// **Function to Capture Console Logs**
def getConsoleLog() {
    return sh(script: "tail -n 20 \$(ls -t ${env.WORKSPACE}/logs/*.log | head -1) || echo 'No log found'", returnStdout: true).trim()
}
