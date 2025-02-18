pipeline {
    agent any
    environment {
        LOG_FILE = 'build_logs.json'  
        BUILD_NUMBER = "${currentBuild.number}"  // Captures build number
        BUILD_START_TIME = "${new Date(currentBuild.getStartTimeInMillis()).format('yyyy-MM-dd HH:mm:ss')}"
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
        always {
            script {
                def metadata = [
                    buildNumber  : env.BUILD_NUMBER,
                    jenkinsUser  : env.BUILD_USER_ID ?: 'Unknown User',
                    startTime    : env.BUILD_START_TIME,
                    endTime      : new Date().format("yyyy-MM-dd HH:mm:ss"),
                    totalDuration: System.currentTimeMillis() - currentBuild.getStartTimeInMillis()
                ]
                updateLogFile(metadata)
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
        updateLogFile([
            stepName  : stepName,
            status    : status,
            durationMs: System.currentTimeMillis() - startTime,
            timestamp : new Date().format("yyyy-MM-dd HH:mm:ss"),
            error     : errorMsg
        ])
    }
}

def updateLogFile(def logEntry) {
    def logFile = "${env.WORKSPACE}/${env.LOG_FILE}"
    def logs = fileExists(logFile) ? readJSON(file: logFile) : []
    logs << logEntry
    writeJSON file: logFile, json: logs
}
