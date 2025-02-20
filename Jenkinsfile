pipeline {
    agent any

    environment {
        DB_URL = 'jdbc:mysql://mysql-container:3306/jenkins_db'
        DB_USER = 'jenkins'
        DB_PASSWORD = 'password'
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    def startTime = new Date()
                    try {
                        git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
                        def endTime = new Date()
                        def duration = endTime.time - startTime.time
                        storeStepData("Clone Repository", "SUCCESS", startTime, endTime, duration)
                    } catch (Exception e) {
                        storeStepData("Clone Repository", "FAILURE", startTime, new Date(), 0)
                        error("Step failed: ${e.message}")
                    }
                }
            }
        }

        stage('Set Up Environment') {
            steps {
                script {
                    captureStepData("Set Up Environment") {
                        echo 'Setting up environment...'
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    captureStepData("Build") {
                        echo 'Building the project...'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    captureStepData("Test") {
                        echo 'Running tests...'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    captureStepData("Deploy") {
                        echo 'Deploying the application...'
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

def captureStepData(stepName, closure) {
    def startTime = new Date()
    try {
        closure()
        def endTime = new Date()
        def duration = endTime.time - startTime.time
        storeStepData(stepName, "SUCCESS", startTime, endTime, duration)
    } catch (Exception e) {
        storeStepData(stepName, "FAILURE", startTime, new Date(), 0)
        error("Step failed: ${e.message}")
    }
}

def storeStepData(stepName, status, startTime, endTime, duration) {
    def buildNumber = env.BUILD_NUMBER
    def jobName = env.JOB_NAME
    def nodeName = env.NODE_NAME ?: "Master"
    def triggeredBy = currentBuild.rawBuild.getCause(hudson.model.Cause$UserIdCause)?.userId ?: "Automated Trigger"
    def gitBranch = env.GIT_BRANCH ?: "main"

    def query = "INSERT INTO step_execution (job_name, build_number, step_name, status, start_time, end_time, duration, triggered_by, git_branch, node_name) VALUES ('${jobName}', ${buildNumber}, '${stepName}', '${status}', '${startTime}', '${endTime}', ${duration}, '${triggeredBy}', '${gitBranch}', '${nodeName}')"

    sh "mysql -h mysql-container -u${DB_USER} -p${DB_PASSWORD} -e \"${query}\" jenkins_db"
}
