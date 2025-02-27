import groovy.json.JsonOutput

pipeline {
    agent any

    environment {
        API_URL = 'https://4ad2-2402-e280-3e1d-bce-6df3-1e62-d8d0-f624.ngrok-free.app/jenkins-metadata'
    }

    stages {
        stage('Initialize Metadata') {
            steps {
                script {
                    METADATA = []  // Initialize the list inside script block
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    try {
                        git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Clone Repository", "SUCCESS", startTime, endTime, "Repository cloned successfully.", METADATA)
                    } catch (Exception e) {
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Clone Repository", "FAILURE", startTime, endTime, e.toString(), METADATA)
                        error("Stage failed: ${e}")
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    try {
                        echo 'Building the project...'
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Build", "SUCCESS", startTime, endTime, "Build completed.", METADATA)
                    } catch (Exception e) {
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Build", "FAILURE", startTime, endTime, e.toString(), METADATA)
                        error("Stage failed: ${e}")
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    try {
                        echo 'Running tests...'
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Test", "SUCCESS", startTime, endTime, "Tests executed successfully.", METADATA)
                    } catch (Exception e) {
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Test", "FAILURE", startTime, endTime, e.toString(), METADATA)
                        error("Stage failed: ${e}")
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    try {
                        echo 'Deploying the application...'
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Deploy", "SUCCESS", startTime, endTime, "Deployment completed successfully.", METADATA)
                    } catch (Exception e) {
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Deploy", "FAILURE", startTime, endTime, e.toString(), METADATA)
                        error("Stage failed: ${e}")
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                sendFinalMetadata(METADATA)
            }
        }
    }
}

// **Function to Append Stage Metadata**
def appendStageMetadata(stageName, status, startTime, endTime, consoleLog, metadataList) {
    def duration = endTime - startTime
    def stepData = [
        stage: stageName,
        status: status,
        startTime: startTime,
        endTime: endTime,
        duration: duration,
        consoleLog: consoleLog
    ]

    metadataList.add(stepData)  // Append to the metadata list
}

// **Function to Send Final Metadata**
def sendFinalMetadata(metadataList) {
    def finalMetadata = [
        metadata: [
            buildNumber: env.BUILD_NUMBER,
            jobName: env.JOB_NAME,
            nodeName: env.NODE_NAME,
            executorNumber: env.EXECUTOR_NUMBER ?: 'N/A',
            buildUrl: env.BUILD_URL
        ],
        steps: metadataList
    ]

    def jsonString = JsonOutput.toJson(finalMetadata)  // Convert to valid JSON format

    sh """curl -X POST ${env.API_URL} \
        -H "Content-Type: application/json" \
        -d '${jsonString.replaceAll("'", "\\'")}'"""  // Escape single quotes for shell compatibility
}
