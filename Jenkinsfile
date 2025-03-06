pipeline {
    agent any

    environment {
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

                    // Fetch Pipeline Data from wfapi
                    def workflowData = sh(script: """curl -s "${env.BUILD_URL}wfapi/describe" """, returnStdout: true).trim()
                    if (!workflowData || workflowData == "null") { error "Failed to fetch wfapi data!" }
                    echo "Extracted Workflow Data: ${workflowData}"

                    // Fetch Build Metadata
                    def buildData = sh(script: """curl -s "${env.BUILD_URL}api/json?tree=number,result,timestamp,estimatedDuration,executor[node,executorUtilization],artifacts[url],url" """, returnStdout: true).trim()
                    if (!buildData || buildData == "null") { error "Failed to fetch build metadata!" }
                    echo "Extracted Build Data: ${buildData}"

                    // Fetch Git Metadata using wfapi
                    def changeSets = sh(script: """curl -s "${env.BUILD_URL}wfapi/changesets" """, returnStdout: true).trim()
                    if (!changeSets || changeSets == "null") { error "Failed to fetch Git data!" }
                    echo "Extracted Git ChangeSets: ${changeSets}"

                    // Prepare JSON Payload
                    def payload = """{
                        "pipeline": ${workflowData},
                        "build": ${buildData},
                        "git": ${changeSets}
                    }"""

                    echo "Formatted Payload: ${payload}"

                    // Send Data to API
                    def response = sh(script: """curl -s -o response.json -w "%{http_code}" -X POST "$API_URL" -H "Content-Type: application/json" --data '${payload}'""", returnStdout: true).trim()
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
