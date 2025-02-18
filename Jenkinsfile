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
                        sh 'echo "Build completed"'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    logStep('Test') {
                        echo 'Running tests...'
                        sh 'echo "Tests passed"'
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
            displayCollectedData()
        }
        failure {
            echo '❌ Pipeline failed!'
            displayCollectedData()
        }
    }
}

def logStep(stepName, Closure body) {
    def startTime = System.currentTimeMillis()
    def status = 'SUCCESS'
    def errorMsg = ''
    def gitData = [
        branch: env.GIT_BRANCH ?: 'N/A',
        commit: env.GIT_COMMIT ?: 'N/A',
        repo  : env.GIT_URL ?: 'N/A'
    ]

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
            stepName      : stepName,
            status        : status,
            durationMs    : duration,
            timestamp     : new Date().format("yyyy-MM-dd HH:mm:ss"),
            error         : errorMsg,
            buildData     : [
                buildNumber: env.BUILD_NUMBER,
                buildId    : env.BUILD_ID,
                jobName    : env.JOB_NAME,
                node       : env.NODE_NAME,
                workspace  : env.WORKSPACE,
                executor   : env.EXECUTOR_NUMBER
            ],
            environment   : env.getEnvironment(),
            repositoryData: gitData
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

def displayCollectedData() {
    def logFile = "${env.WORKSPACE}/${env.LOG_FILE}"

    if (fileExists(logFile)) {
        def logs = readJSON file: logFile
        echo "Collected Data:"
        echo "${groovy.json.JsonOutput.prettyPrint(groovy.json.JsonOutput.toJson(logs))}"
    } else {
        echo "No log data found."
    }
}
