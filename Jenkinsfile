pipeline {
    agent any
    environment {
        LOG_FILE = 'build_logs.json'  
    }
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    logStep('Clone Repository') {
                        git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
                    }
                }
            }
        }

        stage('Set Up Environment') {
            steps {
                script {
                    logStep('Set Up Environment') {
                        echo 'Setting up environment...'
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    logStep('Build') {
                        echo 'Building the project...'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    logStep('Test') {
                        echo 'Running tests...'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    logStep('Deploy') {
                        echo 'Deploying the application...'
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline executed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}

// Function to log step details
def logStep(stepName, Closure body) {
    def startTime = System.currentTimeMillis()
    def status = 'SUCCESS'
    def errorMsg = ''

    try {
        body()
    } catch (Exception e) {
        status = 'FAILED'
        errorMsg = e.getMessage()
        throw e
    } finally {
        def endTime = System.currentTimeMillis()
        def duration = endTime - startTime

        def logEntry = [
            stepName  : stepName,
            status    : status,
            durationMs: duration,
            timestamp : new Date().format("yyyy-MM-dd HH:mm:ss"),
            error     : errorMsg
        ]

        writeLogToFile(logEntry)
    }
}

// Function to write logs to a JSON file using pipeline-safe methods
def writeLogToFile(def logEntry) {
    def logFile = "${env.WORKSPACE}/${env.LOG_FILE}"

    def logs = []
    if (fileExists(logFile)) {
        logs = readJSON file: logFile
    }

    logs << logEntry
    writeJSON file: logFile, json: logs
}
