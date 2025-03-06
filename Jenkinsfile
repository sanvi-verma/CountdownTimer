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

                    // Fetch workflow metadata
                    def workflowData = sh(script: """curl -s "${env.BUILD_URL}wfapi/describe" """, returnStdout: true).trim()
                    def jsonParser = new groovy.json.JsonSlurperClassic()
                    def wfJson = jsonParser.parseText(workflowData)

                    def pipelineData = [
                        "id": wfJson.id ?: "Unknown",
                        "name": wfJson.name ?: "Unknown",
                        "status": wfJson.status ?: "Unknown",
                        "startTime": wfJson.startTimeMillis ?: 0,
                        "endTime": wfJson.endTimeMillis ?: 0,
                        "duration": wfJson.durationMillis ?: 0,
                        "url": env.BUILD_URL,
                        "stages": wfJson.stages.collect { stage -> 
                            [
                                "stage": stage.name,
                                "status": stage.status,
                                "startTime": stage.startTimeMillis ?: 0,
                                "endTime": stage.endTimeMillis ?: 0,
                                "duration": stage.durationMillis ?: 0
                            ]
                        }
                    ]

                    echo "Fetching Build Data..."
                    def buildData = sh(script: """curl -s "${env.BUILD_URL}api/json?depth=1" """, returnStdout: true).trim()
                    def buildJson = jsonParser.parseText(buildData)

                    def build = [
                        "buildNumber": buildJson.id ?: "Unknown",
                        "status": buildJson.inProgress ? "IN_PROGRESS" : (buildJson.result ?: "UNKNOWN"),
                        "triggeredBy": [
                            "userId": buildJson.actions?.find { it.causes }?.causes?.find { it.userId }?.userId ?: "unknown",
                            "userName": buildJson.actions?.find { it.causes }?.causes?.find { it.userName }?.userName ?: "unknown"
                        ],
                        "timestamp": buildJson.timestamp ?: 0,
                        "estimatedDuration": buildJson.estimatedDuration ?: 0,
                        "executor": [
                            "node": "built-in",
                            "executorUtilization": buildJson.actions?.find { it.executorUtilization }?.executorUtilization ?: 1
                        ],
                        "artifactsUrl": "${env.BUILD_URL}artifact",
                        "displayUrl": "${env.BUILD_URL}pipeline-graph",
                        "testsUrl": "${env.BUILD_URL}testReport"
                    ]

                    echo "Fetching Git Data..."
                    def gitData = sh(script: """curl -s "${env.BUILD_URL}api/json?tree=changeSets[items[commitId,author[fullName],authorEmail,msg,date,paths[editType,file]]]" """, returnStdout: true).trim()
                    def gitJson = jsonParser.parseText(gitData)

                    def gitCommits = gitJson.changeSets?.collectMany { it.items } ?: []
                    def git = gitCommits ? [
                        "commitId": gitCommits[0].commitId ?: "Unknown",
                        "affectedFiles": gitCommits[0].paths?.collect { it.file } ?: [],
                        "commitMessage": gitCommits[0].msg ?: "No commit message",
                        "timestamp": gitCommits[0].date ?: "Unknown",
                        "author": [
                            "name": gitCommits[0].author?.fullName ?: "Unknown",
                            "email": gitCommits[0].authorEmail ?: "Unknown"
                        ],
                        "changeType": gitCommits[0].paths?.collect { it.editType } ?: []
                    ] : ["message": "No git data available"]

                    // Create final payload
                    def payload = [
                        "pipeline": pipelineData,
                        "build": build,
                        "git": git
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
