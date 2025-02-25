pipeline {
    agent any

    options {
        timestamps()  // Adds timestamps in logs
        buildDiscarder(logRotator(numToKeepStr: '10'))  // Keep last 10 builds
    }

    environment {
        API_URL = 'https://8ae4-192-245-162-37.ngrok-free.app/jenkins-metadata'
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

def sendMetadata(stageName, status, startTime, endTime) {
    def duration = endTime - startTime
    def buildNumber = env.BUILD_NUMBER
    def jobName = env.JOB_NAME
    def nodeName = env.NODE_NAME
    def executorNumber = env.EXECUTOR_NUMBER ?: "N/A"
    def buildUrl = env.BUILD_URL
    def consoleLog = sh(script: "tail -n 100 ${env.WORKSPACE}/build.log || echo 'No log found'", returnStdout: true).trim()

    def metadata = """{
        "stage": "${stageName}",
        "status": "${status}",
        "startTime": "${startTime}",
        "endTime": "${endTime}",
        "duration": "${duration}",
        "buildNumber": "${buildNumber}",
        "jobName": "${jobName}",
        "nodeName": "${nodeName}",
        "executorNumber": "${executorNumber}",
        "buildUrl": "${buildUrl}",
        "consoleLog": "${consoleLog.replaceAll("\"", "'")}"
    }"""

    sh """curl -X POST ${API_URL} \
        -H "Content-Type: application/json" \
        -d '${metadata}'"""
}
