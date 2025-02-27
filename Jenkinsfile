pipeline {
    agent any

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        API_URL = 'https://4ad2-2402-e280-3e1d-bce-6df3-1e62-d8d0-f624.ngrok-free.app/jenkins-metadata'
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    executeStage("Clone Repository") {
                        git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    executeStage("Build") {
                        echo 'Compiling source files...'
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                sendFinalMetadata()
            }
        }
    }
}

// Function to execute stage and send metadata
def executeStage(stageName, Closure body) {
    def startTime = System.currentTimeMillis()
    try {
        body.call()
        def endTime = System.currentTimeMillis()
        saveStageMetadata(stageName, "SUCCESS", startTime, endTime)
    } catch (Exception e) {
        def endTime = System.currentTimeMillis()
        saveStageMetadata(stageName, "FAILURE", startTime, endTime)
        error("Stage failed: ${e}")
    }
}

// Store step metadata
def stepsMetadata = []

def saveStageMetadata(stageName, status, startTime, endTime) {
    def duration = endTime - startTime
    def consoleLog = sh(script: "echo '${stageName} completed successfully.'", returnStdout: true).trim()

    stepsMetadata.add([
        stage: stageName,
        status: status,
        startTime: startTime.toString(),
        endTime: endTime.toString(),
        duration: duration.toString(),
        consoleLog: consoleLog
    ])
}

// Send final metadata
def sendFinalMetadata() {
    def metadata = [
        pipeline: env.JOB_NAME,
        buildNumber: env.BUILD_NUMBER,
        jobName: env.JOB_NAME,
        nodeName: env.NODE_NAME,
        executorNumber: env.EXECUTOR_NUMBER ?: "N/A",
        buildUrl: env.BUILD_URL,
        timestamp: new Date().format("yyyy-MM-dd'T'HH:mm:ss'Z'", TimeZone.getTimeZone('UTC')),
        jenkinsMeta: [
            jenkinsHome: env.JENKINS_HOME,
            jenkinsUrl: env.JENKINS_URL,
            gitCommit: sh(script: "git rev-parse HEAD", returnStdout: true).trim(),
            branchName: sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim(),
            buildTimestamp: new Date().format("yyyy-MM-dd'T'HH:mm:ss'Z'", TimeZone.getTimeZone('UTC')),
            buildCause: currentBuild.getBuildCauses().toString(),
            workspace: env.WORKSPACE,
            buildUser: env.BUILD_USER ?: "Unknown"
        ],
        steps: stepsMetadata
    ]

    sh """curl -X POST ${API_URL} \
        -H "Content-Type: application/json" \
        -d '${groovy.json.JsonOutput.toJson(metadata)}'"""
}
