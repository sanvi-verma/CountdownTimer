pipeline {
    agent any
    environment {
        LOG_FILE = 'build_logs.json'  
    }
    tools
    {
        git 'git'
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


def writeLogToFile(def logEntry) {
    def logFile = new File(env.WORKSPACE, env.LOG_FILE)
    def logs = []

    if (logFile.exists()) {
        logs = readJSON file: env.LOG_FILE
    }
    
    logs << logEntry
    writeJSON file: env.LOG_FILE, json: logs
}
