import groovy.json.JsonOutput

pipeline {
    agent any

    environment {
        JENKINS_URL = "http://localhost:8080"
        API_URL = "https://2cf9-2402-e280-3e1d-bce-2584-894f-4e39-6c7c.ngrok-free.app/jenkins-metadata"
        API_KEY = 'qteyew2537e3ygdhusdhd833'
    }

    stages {
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
    }

    post {
        always {
            script {
                collectAndSendWFAPI()
            }
        }
    }
}

def collectAndSendWFAPI() {
    try {
        // Fetch WFAPI data
        def wfapiUrl = "${env.JENKINS_URL}/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/wfapi/runs"
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

        // Extract Steps
        def steps = wfapiData.collect { step ->
            [
                stage: step.name,
                status: step.status,
                startTime: step.startTimeMillis,
                endTime: step.startTimeMillis + step.durationMillis,
                duration: step.durationMillis,
                consoleLog: getConsoleLog(step.id) // Fetch console log
            ]
        }

        // Final JSON Payload
        def jsonPayload = JsonOutput.toJson([data: [metadata: metadata, steps: steps]])

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
