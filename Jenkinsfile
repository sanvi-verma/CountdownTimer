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

                    // Fetch pipeline metadata dynamically
                    def pipelineData = sh(script: """curl -s "${env.BUILD_URL}wfapi/describe" """, returnStdout: true).trim()
                    
                    if (!pipelineData || pipelineData == "null") {
                        error "Failed to fetch pipeline data!"
                    }

                    echo "Extracted Pipeline Data: ${pipelineData}"

                    // Fetch build metadata dynamically
                    echo "Fetching Build Data..."
                    def buildData = sh(script: """curl -s "${env.BUILD_URL}api/json?depth=1" """, returnStdout: true).trim()

                    // Fetch Git data dynamically
                    echo "Fetching Git Data..."
                    def gitData = sh(script: """curl -s "${env.BUILD_URL}api/json?tree=changeSets[items[commitId,author[fullName],authorEmail,msg,date,paths[editType,file]]]" """, returnStdout: true).trim()

                    // Create final payload dynamically
                    def payloadJson = """{
                        "pipeline": ${pipelineData},
                        "build": ${buildData},
                        "git": ${gitData}
                    }"""

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
