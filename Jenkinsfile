pipeline {
    agent any

    environment {
        JENKINS_URL = "http://localhost:8080"
        API_URL = "https://2cf9-2402-e280-3e1d-bce-2584-894f-4e39-6c7c.ngrok-free.app/jenkins-metadata"
         API_KEY = "qteyew2537e3ygdhusdhd833"
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
                echo 'Building the project...'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying the application...'
            }
        }

        stage('Fetch Real-Time Data') {
            steps {
                script {
                    echo "Fetching Workflow Data from wfapi..."

                    def workflowData = sh(script: """curl -s "$JENKINS_URL${env.BUILD_URL}wfapi/describe" """, returnStdout: true).trim()
                    
                    if (!workflowData || workflowData == "null") {
                        error "Failed to fetch wfapi data!"
                    }

                    echo "Extracted Workflow Data: ${workflowData}"

                    // Parse JSON data using groovy.json.JsonSlurper
                    def jsonParser = new groovy.json.JsonSlurper()
                    def wfJson = jsonParser.parseText(workflowData)

                    // Extract pipeline metadata
                    def pipelineData = [
                        "name": wfJson.name ?: "Unknown",
                        "status": wfJson.status ?: "Unknown",
                        "durationMillis": wfJson.durationMillis ?: 0,
                        "stages": wfJson.stages ?: []
                    ]

                    echo "Fetching Build Data..."
                    def buildData = sh(script: """curl -s "$JENKINS_URL${env.BUILD_URL}api/json?depth=1" """, returnStdout: true).trim()
                    
                    echo "Fetching Git Data..."
                    def gitData = sh(script: """curl -s "$JENKINS_URL${env.BUILD_URL}api/json?tree=changeSets[items[commitId,author[fullName],authorEmail,msg,date,paths[editType,file]]]" """, returnStdout: true).trim()

                    // Create final payload
                    def payload = [
                        "pipelineData": pipelineData,
                        "buildData": buildData ?: "{}",
                        "gitData": gitData ?: "[]"
                    ]

                    def payloadJson = new groovy.json.JsonBuilder(payload).toString()
                    echo "Formatted Payload: ${payloadJson}"

                    def response = sh(script: """curl -s -o response.json -w "%{http_code}" -X POST "$API_URL" -H "Content-Type: application/json" --data '${payloadJson}'""", returnStdout: true).trim()
                    def httpStatus = response[-3..-1]

                    if (httpStatus != "200") {
                        error "Failed to send data to API, HTTP Status: ${httpStatus}"
                    } else {
                        echo "Data successfully sent to API."
                    }
                }
            }
        }
    }
}
