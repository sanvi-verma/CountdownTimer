pipeline {
    agent any

    environment {
        DB_HOST = "host.docker.internal"
        DB_NAME = "jenkins_db"
        DB_USER = "root"
        DB_PASSWORD = "root"
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
                    echo "Cloning repository..."
                    saveStepToDB("Clone Repository", "SUCCESS")
                }
            }
        }

        stage('Set Up Environment') {
            steps {
                script {
                    echo "Setting up environment..."
                    saveStepToDB("Set Up Environment", "SUCCESS")
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building..."
                    saveStepToDB("Build", "SUCCESS")
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo "Running tests..."
                    saveStepToDB("Test", "SUCCESS")
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Deploying..."
                    saveStepToDB("Deploy", "SUCCESS")
                }
            }
        }
    }

    post {
        always {
            script {
                def endTime = new Date().format("yyyy-MM-dd HH:mm:ss")
                def duration = System.currentTimeMillis() - currentBuild.getStartTimeInMillis()

                sh """#!/bin/bash
                mysql --user="$DB_USER" --password="$DB_PASSWORD" --host="$DB_HOST" --database="$DB_NAME" --execute="
                INSERT INTO step_execution (job_name, build_number, step_name, status, start_time, end_time, duration, triggered_by, git_branch, node_name) 
                VALUES ('pipeline-job', '$BUILD_NUMBER', 'Pipeline Completed', 'SUCCESS', '$env.START_TIME', '$endTime', '$duration', 'Unknown User', 'origin/main', 'built-in');
                "
                """
            }
        }
    }
}

def saveStepToDB(stepName, status) {
    def startTime = new Date().format("yyyy-MM-dd HH:mm:ss")
    def duration = System.currentTimeMillis() - currentBuild.getStartTimeInMillis()

    sh """#!/bin/bash
    mysql --user="$DB_USER" --password="$DB_PASSWORD" --host="$DB_HOST" --database="$DB_NAME" --execute="
    INSERT INTO step_execution (job_name, build_number, step_name, status, start_time, end_time, duration, triggered_by, git_branch, node_name) 
    VALUES ('pipeline-job', '$BUILD_NUMBER', '$stepName', '$status', '$startTime', NOW(), '$duration', 'Unknown User', 'origin/main', 'built-in');
    "
    """
}
