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
                        echo "Cloning repository..."
                    }
                }
            }
        }

        stage('Set Up Environment') {
            steps {
                script {
                    logStep("Set Up Environment") {
                        echo "Setting up environment..."
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    logStep("Build") {
                        echo "Building..."
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    logStep("Test") {
                        echo "Running tests..."
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    logStep("Deploy") {
                        echo "Deploying..."
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // Debug: Print workspace path
                echo "Checking workspace: ${env.WORKSPACE}"
                sh "ls -la ${env.WORKSPACE}"

                def logFile = "${env.WORKSPACE}/${env.LOG_FILE}"

                // Debug: Check if the log file exists
                if (fileExists(logFile)) {
                    echo "✅ build_logs.json exists!"
                    sh "cat ${logFile}"
                } else {
                    echo "❌ build_logs.json NOT found! Creating an empty file..."
                    writeJSON file: logFile, json: []
                }

                // Add build metadata to the log
                def endTime = new Date().format("yyyy-MM-dd HH:mm:ss")
                def duration = System.currentTimeMillis() - currentBuild.getStartTimeInMillis()

                def metadata = [
                    buildNumber  : env.BUILD_NUMBER,
                    jenkinsUser  : env.BUILD_USER_ID ?: 'Unknown User',
                    startTime    : env.START_TIME,
                    endTime      : endTime,
                    totalDuration: duration
                ]

                // Append metadata to the log file
                def logs = []
                if (fileExists(logFile)) {
                    logs = readJSON file: logFile
                }
                logs << metadata
                writeJSON file: logFile, json: logs

                echo "📄 build_logs.json updated successfully!"
            }
        }
    }
}

// Function to log each step
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

// Function to write logs to file
def writeLogToFile(def logEntry) {
    def logFile = "${env.WORKSPACE}/${env.LOG_FILE}"
    def logs = []
    if (fileExists(logFile)) {
        logs = readJSON file: logFile
    }

    logs << logEntry
    writeJSON file: logFile, json: logs
}
