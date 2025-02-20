pipeline {
    agent any

    environment {
        DB_HOST = "host.docker.internal"
        DB_NAME = "jenkins_db"
    }

    stages {
        stage('Start Build') {
            steps {
                script {
                    env.START_TIME = new Date().format("yyyy-MM-dd HH:mm:ss")
                    echo "Build started at ${env.START_TIME}"
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'mysql-credentials', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASSWORD')]) {
                        logStep("Clone Repository") {
                            echo "Cloning repository..."
                            saveStepToDB("Clone Repository", "SUCCESS")
                        }
                    }
                }
            }
        }

        stage('Set Up Environment') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'mysql-credentials', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASSWORD')]) {
                        logStep("Set Up Environment") {
                            echo "Setting up environment..."
                            saveStepToDB("Set Up Environment", "SUCCESS")
                        }
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'mysql-credentials', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASSWORD')]) {
                        logStep("Build") {
                            echo "Building..."
                            saveStepToDB("Build", "SUCCESS")
                        }
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'mysql-credentials', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASSWORD')]) {
                        logStep("Test") {
                            echo "Running tests..."
                            saveStepToDB("Test", "SUCCESS")
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'mysql-credentials', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASSWORD')]) {
                        logStep("Deploy") {
                            echo "Deploying..."
                            saveStepToDB("Deploy", "SUCCESS")
                        }
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

                withCredentials([usernamePassword(credentialsId: 'mysql-credentials', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASSWORD')]) {
    sh '''#!/bin/bash
    mysql --user="$DB_USER" --password="$DB_PASSWORD" --host="host.docker.internal" --database="jenkins_db" --execute="
    INSERT INTO step_execution (job_name, build_number, step_name, status, start_time, end_time, duration, triggered_by, git_branch, node_name) 
    VALUES ('portfolio-pipeline', '$BUILD_NUMBER', '$stepName', '$status', '$startTime', NOW(), '$duration', 'Unknown User', 'origin/main', 'built-in');
    "
    '''


                }
            }
        }
    }
}
def saveStepToDB(stepName, status) {
    def startTime = new Date().format("yyyy-MM-dd HH:mm:ss")
    def duration = System.currentTimeMillis() - currentBuild.getStartTimeInMillis()
    
    withCredentials([usernamePassword(credentialsId: 'mysql-credentials', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASSWORD')]) {
        sh """#!/bin/bash
        export STEP_NAME="${stepName}"
        export STEP_STATUS="${status}"
        export START_TIME="${startTime}"
        export DURATION="${duration}"

        mysql --user="$DB_USER" --password="$DB_PASSWORD" --host="host.docker.internal" --database="jenkins_db" --execute="
        INSERT INTO step_execution (job_name, build_number, step_name, status, start_time, end_time, duration, triggered_by, git_branch, node_name) 
        VALUES ('portfolio-pipeline', '$BUILD_NUMBER', '$STEP_NAME', '$STEP_STATUS', '$START_TIME', NOW(), '$DURATION', 'Unknown User', 'origin/main', 'built-in');
        "
        """
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
    } finally {
        def endTime = System.currentTimeMillis()
        def duration = endTime - startTime
        def timestamp = new Date().format("yyyy-MM-dd HH:mm:ss")

        saveStepToDB(stepName, status)
        
        if (status == 'FAILED') {
            throw new Exception("Step ${stepName} failed: ${errorMsg}")
        }
    }
}

