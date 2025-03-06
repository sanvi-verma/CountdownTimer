import groovy.json.JsonOutput

pipeline {
    agent any

    environment {
        JENKINS_URL = "http://localhost:8080"
        API_URL = "https://2cf9-2402-e280-3e1d-bce-2584-894f-4e39-6c7c.ngrok-free.app/jenkins-metadata"
        API_KEY = 'qteyew2537e3ygdhusdhd833'
    }

    stages {
        stage('Declarative: Checkout SCM') {
            steps {
                script {
                    echo 'Checking out source code from SCM...'
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    echo 'Repository cloned successfully.'
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo 'Build completed.'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo 'Tests executed successfully.'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo 'Deployment completed successfully.'
                }
            }
        }

        stage('Declarative: Post Actions') {
            steps {
                script {
                    echo 'Executing post-build actions...'
                }
            }
        }
    }

    post {
        always {
            script {
                collectAndSendWFAPI()
            }
        }
    }
}

// Function to collect and send metadata using WFAPI
def collectAndSendWFAPI() {
    try {
        def wfapiUrl = "${env.JENKINS_URL}/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/wfapi/describe"
        echo "Fetching pipeline metadata from: ${wfapiUrl}"

        def response = httpRequest(
            httpMode: 'GET',
            url: wfapiUrl,
            contentType: 'APPLICATION_JSON'
        )

        def wfapiData = readJSON(text: response.content)

        // Extract Metadata
        def metadata = [
            buildNumber: env.BUILD_NUMBER,
            jobName: env.JOB_NAME,
            nodeName: env.NODE_NAME ?: "built-in",
            executorNumber: env.EXECUTOR_NUMBER ?: "N/A",
            buildUrl: "${env.JENKINS_URL}/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/"
        ]

        // Define required stages to extract only necessary ones
        def requiredStages = [
            "Declarative: Checkout SCM",
            "Clone Repository",
            "Build",
            "Test",
            "Deploy",
            "Declarative: Post Actions"
        ]

        // Extract Steps only for required stages
        def steps = wfapiData.stages.findAll { stage -> stage.name in requiredStages }.collect { stage ->
            [
                stage: stage.name,
                status: stage.status,
                startTime: stage.startTimeMillis,
                endTime: stage.startTimeMillis + stage.durationMillis,
                duration: stage.durationMillis,
                consoleLog: getConsoleLog(stage.id)
            ]
        }

        // Final JSON Payload
        def jsonPayload = JsonOutput.toJson([metadata: metadata, steps: steps])

        // Send data to external API
        def postResponse = httpRequest(
            httpMode: 'POST',
            url: "${env.API_URL}",
            requestBody: jsonPayload,
            contentType: 'APPLICATION_JSON',
            customHeaders: [[name: 'Authorization', value: "Bearer ${env.API_KEY}"]]
        )

        echo "Metadata successfully sent to API: ${postResponse.content}"

    } catch (Exception e) {
        echo "Failed to fetch/send WFAPI data: ${e}"
    }
}

// Function to fetch console logs
def getConsoleLog(stageId) {
    try {
        def consoleLogUrl = "${env.JENKINS_URL}/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/execution/node/${stageId}/logText/progressiveText"
        def logResponse = httpRequest(httpMode: 'GET', url: consoleLogUrl)
        return logResponse.content?.trim() ?: "No logs available."
    } catch (Exception e) {
        return "Error fetching logs: ${e.message}"
    }
}
