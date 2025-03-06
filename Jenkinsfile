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
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER

            echo "Fetching Pipeline Data..."
            def pipelineData = sh(script: """curl -s "$JENKINS_URL/job/$jobName/$buildNumber/wfapi/describe" """, returnStdout: true).trim()
            echo "Pipeline Data: ${pipelineData}"

            echo "Fetching Build Data..."
            def buildData = sh(script: """curl -s "$JENKINS_URL/job/$jobName/$buildNumber/api/json?depth=1" """, returnStdout: true).trim()
            echo "Build Data: ${buildData}"

            echo "Fetching Git Data..."
            def gitData = sh(script: """curl -s "$JENKINS_URL/job/$jobName/$buildNumber/api/json?tree=changeSets[items[commitId,author[fullName],authorEmail,msg,date,paths[editType,file]]]" """, returnStdout: true).trim()
            echo "Git Data: ${gitData}"

            // Ensure JSON format is valid
            def payload = """{
                "pipelineData": ${pipelineData ?: '{}'},
                "buildData": ${buildData ?: '{}'},
                "gitData": ${gitData ?: '[]'}
            }"""

            echo "Formatted Payload: ${payload}"

            sh """curl -X POST "$API_URL" -H "Content-Type: application/json" --data '${payload}'"""
        }
    }
}

    }
}
