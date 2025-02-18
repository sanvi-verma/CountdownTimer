pipeline {
    agent any

    environment {
        LOG_FILE = 'build_logs.json'
    }

    stages {
        stage('Start Build') {
            steps {
                script {
                    def startTime = new Date().format("yyyy-MM-dd HH:mm:ss")
                    env.START_TIME = startTime
                    echo "Build started at ${startTime}"
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    logStep("Clone Repository") {
                        // Your clone repository command here
                        echo "Cloning repository..."
                    }
                }
            }
        }

        stage('Set Up Environment') {
            steps {
                script {
                    logStep("Set Up Environment") {
                        // Your setup environment command here
                        echo "Setting up environment..."
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    logStep("Build") {
                        // Your build command here
                        echo "Building..."
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    logStep("Test") {
                        // Your test command here
                        echo "Running tests..."
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    logStep("Deploy") {
                        // Your deploy command here
                        echo "Deploying..."
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // Add build metadata to the log
                def endTime = new Date().format("yyyy-MM-dd HH:mm:ss")
                def duration = System.currentTimeMillis() - currentBuild.getStartTimeInMillis()

                def metadata = [
                    buildNumber: env.BUILD_NUMBER,
                    jenkinsUser: env.BUILD_USER_ID ?: 'Unknown User',
                    startTime: env.START_TIME,
                    endTime: endTime,
                    totalDuration: duration
                ]

                // Write metadata to the log file
                def logFile = "${env.WORKSPACE}/${env.LOG_FILE}"
                def logs = []
                if (fileExists(logFile)) {
                    logs = readJSON file: logFile
                }
                logs << metadata
                writeJSON file: logFile, json: logs
            }
        }
    }
}

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
        def timestamp = new Date().format("yyyy-MM-dd HH:mm:ss")

        def logEntry = [
            stepName  : stepName,
            status    : status,
            durationMs: duration,
            timestamp : timestamp,
            error     : errorMsg
        ]

        writeLogToFile(logEntry)
    }
}

def writeLogToFile(def logEntry) {
    def logFile = "${env.WORKSPACE}/${env.LOG_FILE}"
    def logs = []
    if (fileExists(logFile)) {
        logs = readJSON file: logFile
    }

    logs << logEntry
    writeJSON file: logFile, json: logs
}
