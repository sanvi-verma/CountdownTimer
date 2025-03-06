pipeline {
    agent any

    environment {
        JENKINS_URL = "http://localhost:8080"
        API_URL = "https://2cf9-2402-e280-3e1d-bce-2584-894f-4e39-6c7c.ngrok-free.app/jenkins-metadata"
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo 'Building the project...'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo 'Running tests...'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo 'Deploying the application...'
                }
            }
        }

       stage('Fetch Real-Time Data') {
    steps {
        script {
            def jobName = env.JOB_NAME ?: 'UNKNOWN_JOB'
            def buildNumber = env.BUILD_NUMBER ?: 'UNKNOWN_BUILD'

            echo "Resolved JOB_NAME: ${jobName}"
            echo "Resolved BUILD_NUMBER: ${buildNumber}"

            def apiUrl = "$JENKINS_URL/job/${jobName}/${buildNumber}/wfapi/describe"

            echo "API URL: ${apiUrl}"

            def pipelineRaw = sh(script: """curl -s "${apiUrl}" """, returnStdout: true).trim()

            if (!pipelineRaw || pipelineRaw == "{}") {
                error("Error: No pipeline data fetched!")
            }

            echo "Pipeline Raw Data: ${pipelineRaw}"

            def pipelineJson = readJSON text: pipelineRaw
        }
    }
}
    }
}
