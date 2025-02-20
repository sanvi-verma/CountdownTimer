pipeline {
    agent any

    environment {
        DB_HOST = "host.docker.internal"
        DB_NAME = "jenkins_db"
        DB_USER = "jenkins"
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
                    withCredentials([string(credentialsId: 'mysql-password', variable: 'DB_PASSWORD')]) {
                        echo "Cloning repository..."
                        sh """
                        mysql --user=$DB_USER --password=$DB_PASSWORD --host=$DB_HOST --database=$DB_NAME --execute="
                        INSERT INTO step_execution (job_name, build_number, step_name, status, start_time, end_time, duration, triggered_by, git_branch, node_name) 
                        VALUES ('portfolio-pipeline', ${env.BUILD_NUMBER}, 'Clone Repository', 'SUCCESS', NOW(), NOW(), 0, 'Unknown User', 'origin/main', 'built-in')"
                        """
                    }
                }
            }
        }

        stage('Set Up Environment') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'mysql-password', variable: 'DB_PASSWORD')]) {
                        echo "Setting up environment..."
                        sh """
                        mysql --user=$DB_USER --password=$DB_PASSWORD --host=$DB_HOST --database=$DB_NAME --execute="
                        INSERT INTO step_execution (job_name, build_number, step_name, status, start_time, end_time, duration, triggered_by, git_branch, node_name) 
                        VALUES ('portfolio-pipeline', ${env.BUILD_NUMBER}, 'Set Up Environment', 'SUCCESS', NOW(), NOW(), 0, 'Unknown User', 'origin/main', 'built-in')"
                        """
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'mysql-password', variable: 'DB_PASSWORD')]) {
                        echo "Building..."
                        sh """
                        mysql --user=$DB_USER --password=$DB_PASSWORD --host=$DB_HOST --database=$DB_NAME --execute="
                        INSERT INTO step_execution (job_name, build_number, step_name, status, start_time, end_time, duration, triggered_by, git_branch, node_name) 
                        VALUES ('portfolio-pipeline', ${env.BUILD_NUMBER}, 'Build', 'SUCCESS', NOW(), NOW(), 0, 'Unknown User', 'origin/main', 'built-in')"
                        """
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'mysql-password', variable: 'DB_PASSWORD')]) {
                        echo "Running tests..."
                        sh """
                        mysql --user=$DB_USER --password=$DB_PASSWORD --host=$DB_HOST --database=$DB_NAME --execute="
                        INSERT INTO step_execution (job_name, build_number, step_name, status, start_time, end_time, duration, triggered_by, git_branch, node_name) 
                        VALUES ('portfolio-pipeline', ${env.BUILD_NUMBER}, 'Test', 'SUCCESS', NOW(), NOW(), 0, 'Unknown User', 'origin/main', 'built-in')"
                        """
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'mysql-password', variable: 'DB_PASSWORD')]) {
                        echo "Deploying..."
                        sh """
                        mysql --user=$DB_USER --password=$DB_PASSWORD --host=$DB_HOST --database=$DB_NAME --execute="
                        INSERT INTO step_execution (job_name, build_number, step_name, status, start_time, end_time, duration, triggered_by, git_branch, node_name) 
                        VALUES ('portfolio-pipeline', ${env.BUILD_NUMBER}, 'Deploy', 'SUCCESS', NOW(), NOW(), 0, 'Unknown User', 'origin/main', 'built-in')"
                        """
                    }
                }
            }
        }
    }
}
