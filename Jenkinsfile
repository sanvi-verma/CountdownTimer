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
                    echo "Fetching Workflow Data from wfapi..."

                    // Fetch Workflow Data from wfapi
                    def pipelineData = sh(script: """curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/wfapi/describe" """, returnStdout: true).trim()
                    if (!pipelineData || pipelineData == "null") { error "Failed to fetch wfapi data!" }
                    echo "Extracted Pipeline Data: ${pipelineData}"

                    // Fetch Build Metadata from Jenkins API
                    def buildData = sh(script: """curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/api/json?tree=number,result,timestamp,estimatedDuration,executor[node,executorUtilization],artifacts[url],url" """, returnStdout: true).trim()
                    if (!buildData || buildData == "null") { error "Failed to fetch build metadata!" }
                    echo "Extracted Build Data: ${buildData}"

                    // Fetch Git Metadata (Branch, Commit, Changes) from wfapi
                    def changeSetsData = sh(script: """curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/wfapi/changesets" """, returnStdout: true).trim()
                    if (!changeSetsData || changeSetsData == "null") { error "Failed to fetch Git data!" }
                    echo "Extracted Git ChangeSets: ${changeSetsData}"

                    // Parse JSON to extract Git Branch & Commit dynamically
                    def gitData = readJSON(text: changeSetsData)
                    def gitBranch = gitData[0]?.branch ?: "unknown"
                    def gitCommit = gitData[0]?.commitId ?: "unknown"

                    // Merge All Data into a Single JSON Object
                    def payload = """{
                        "pipeline": ${pipelineData},
                        "build": ${buildData},
                        "git": {
                            "branch": "${gitBranch}",
                            "commit": "${gitCommit}",
                            "changes": ${changeSetsData}
                        }
                    }"""

                    echo "Complete Metadata: ${payload}"

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
