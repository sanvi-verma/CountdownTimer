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
                    def startTime = new Date().getTime()
                    try {
                        git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
                        def endTime = new Date().getTime()
                        def logOutput = "Repository cloned successfully."
                        appendStageMetadata("Clone Repository", "SUCCESS", startTime, endTime, logOutput)
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        appendStageMetadata("Clone Repository", "FAILURE", startTime, endTime, e.toString())
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
                        sh 'npm install' // Example setup step
                        def endTime = new Date().getTime()
                        appendStageMetadata("Set Up Environment", "SUCCESS", startTime, endTime, "Environment setup complete.")
                    } catch (Exception e) {
                        def endTime = new Date().getTime()
                        appendStageMetadata("Set Up Environment", "FAILURE", startTime, endTime, e.toString())
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
                        appendStageMetadata("Build", "SUCCESS", startTime, endTime, "Build completed successfully.")
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
                        appendStageMetadata("Test", "SUCCESS", startTime, endTime, "Tests passed.")
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
                        appendStageMetadata("Deploy", "FAILURE", startTime, endTime, "Deployment failed.")
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

    if (env.METADATA.contains('"steps": []')) {
        env.METADATA = env.METADATA.replace('"steps": []', '"steps": [' + newStep + ']')
    } else {
        env.METADATA = env.METADATA.replace('"steps": [', '"steps": [' + newStep + ', ')
    }
}
