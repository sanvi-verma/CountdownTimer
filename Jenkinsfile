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
        stage('Initialize Metadata') {
            steps {
                script {
                    env.METADATA = """{
                        "metadata": {
                            "buildNumber": "${env.BUILD_NUMBER}",
                            "jobName": "${env.JOB_NAME}",
                            "nodeName": "${env.NODE_NAME}",
                            "executorNumber": "${env.EXECUTOR_NUMBER ?: 'N/A'}",
                            "buildUrl": "${env.BUILD_URL}"
                        },
                        "steps": []
                    }"""
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    def startTime = new Date().getTime()
                    try {
                        git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
                        def endTime = new Date().getTime()
                        def logOutput = sh(script: "tail -n 10 ${env.WORKSPACE}/build.log || echo 'No log found'", returnStdout: true).trim()
                        appendStageMetadata("Clone Repository", "SUCCESS", startTime, endTime, logOutput)
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        appendStageMetadata("Clone Repository", "FAILURE", startTime, endTime, e.toString())
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
                        appendStageMetadata("Build", "SUCCESS", startTime, endTime, "Compiling source code...")
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        appendStageMetadata("Build", "FAILURE", startTime, endTime, e.toString())
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
                        appendStageMetadata("Test", "SUCCESS", startTime, endTime, "Running tests...")
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        appendStageMetadata("Test", "FAILURE", startTime, endTime, e.toString())
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
                        appendStageMetadata("Deploy", "SUCCESS", startTime, endTime, "Deployment completed successfully.")
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        appendStageMetadata("Deploy", "FAILURE", startTime, endTime, "Deployment failed due to missing config.")
                        error("Stage failed: ${e}")
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                sh """curl -X POST ${API_URL} \
                    -H "Content-Type: application/json" \
                    -d '${env.METADATA}'"""
            }
        }
    }
}

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

    env.METADATA = env.METADATA.replaceFirst('"steps": \\[', '"steps": [' + newStep + (env.METADATA.contains('"steps": []') ? "" : ", "))
}
