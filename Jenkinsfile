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
                    def startTime = System.currentTimeMillis()
                    try {
                        git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
                        
                        // Get commit SHA
                        def commitHash = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                        
                        def endTime = System.currentTimeMillis()
                        sendMetadata("Clone Repository", "SUCCESS", startTime, endTime, commitHash)
                    } catch (Exception e) {
                        def endTime = System.currentTimeMillis()
                        sendMetadata("Clone Repository", "FAILURE", startTime, endTime, "N/A", e.toString())
                        error("Stage failed: ${e}")
                    }
                }
            }
        }

        stage('Set Up Environment') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    try {
                        echo 'Setting up environment...'
                        def endTime = System.currentTimeMillis()
                        sendMetadata("Set Up Environment", "SUCCESS", startTime, endTime)
                    } catch (Exception e) {
                        def endTime = System.currentTimeMillis()
                        sendMetadata("Set Up Environment", "FAILURE", startTime, endTime, "N/A", e.toString())
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
                        sendMetadata("Build", "SUCCESS", startTime, endTime)
                    } catch (Exception e) {
                        def endTime = System.currentTimeMillis()
                        sendMetadata("Build", "FAILURE", startTime, endTime, "N/A", e.toString())
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
                        sendMetadata("Test", "SUCCESS", startTime, endTime)
                    } catch (Exception e) {
                        def endTime = System.currentTimeMillis()
                        sendMetadata("Test", "FAILURE", startTime, endTime, "N/A", e.toString())
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
                        sendMetadata("Deploy", "SUCCESS", startTime, endTime)
                    } catch (Exception e) {
                        def endTime = System.currentTimeMillis()
                        sendMetadata("Deploy", "FAILURE", startTime, endTime, "N/A", e.toString())
                        error("Stage failed: ${e}")
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                sendMetadata("Pipeline", "SUCCESS", 0, 0)
                echo 'Pipeline executed successfully!'
            }
        }
        failure {
            script {
                sendMetadata("Pipeline", "FAILURE", 0, 0)
                echo 'Pipeline failed!'
            }
        }
    }
}

def sendMetadata(stageName, status, startTime, endTime, commitHash = "N/A", errorMessage = "None") {
    def duration = endTime - startTime
    def buildNumber = env.BUILD_NUMBER ?: "Unknown"
    def jobName = env.JOB_NAME ?: "Unknown"
    def nodeName = env.NODE_NAME ?: "Unknown"
    def executorNumber = env.EXECUTOR_NUMBER ?: "N/A"
    def buildUrl = env.BUILD_URL ?: "Unknown"
    def user = env.BUILD_USER ?: "Automated"
    def workspace = env.WORKSPACE ?: "Unknown"

    // Get system CPU and memory usage
    def systemStats = sh(script: "top -b -n1 | head -n5", returnStdout: true).trim()

    // Convert timestamps to readable format
    def startTimestamp = new Date(startTime).toString()
    def endTimestamp = new Date(endTime).toString()

    def metadata = """{
        "stage": "${stageName}",
        "status": "${status}",
        "startTime": "${startTime}",
        "startTimestamp": "${startTimestamp}",
        "endTime": "${endTime}",
        "endTimestamp": "${endTimestamp}",
        "duration": "${duration}",
        "buildNumber": "${buildNumber}",
        "jobName": "${jobName}",
        "nodeName": "${nodeName}",
        "executorNumber": "${executorNumber}",
        "buildUrl": "${buildUrl}",
        "commitHash": "${commitHash}",
        "workspace": "${workspace}",
        "triggeredBy": "${user}",
        "systemStats": "${systemStats.replaceAll("\"", "'")}",
        "errorMessage": "${errorMessage}"
    }"""

    sh """curl -X POST ${API_URL} \
        -H "Content-Type: application/json" \
        -d '${metadata}'"""
}
