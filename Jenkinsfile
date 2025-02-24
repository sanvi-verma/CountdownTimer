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
                def endTime = new Date().format("yyyy-MM-dd HH:mm:ss")
                def duration = System.currentTimeMillis() - currentBuild.getStartTimeInMillis()

                def metadata = [
                    buildNumber: env.BUILD_NUMBER,
                    jenkinsUser: env.BUILD_USER_ID ?: 'Unknown User',
                    startTime: env.START_TIME,
                    endTime: endTime,
                    totalDuration: duration
                ]

                writeLogToFile(metadata)
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
        publishToRabbitMQ(logEntry)
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

def publishToRabbitMQ(def logEntry) {
    def message = groovy.json.JsonOutput.toJson(logEntry)

    rabbitmqPublisher(
        name: 'rabbitmq-broker',        // Configured RabbitMQ broker in Jenkins
        hostname: 'rabbitmq',           // RabbitMQ container hostname
        portNumber: '5672',             // RabbitMQ port
        virtualHost: '/',               // Virtual host
        topic: 'jenkins-logs',          // Routing key
        exchange: 'jenkins-exchange',   // Exchange name
        queue: 'jenkins-logs',          // Queue name
        authenticationMethod: 'UsernamePassword', // Authentication method
        message: message                // Log entry as JSON string
    )
}
