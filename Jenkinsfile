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

                    // Fetch Pipeline Metadata
                    def pipelineData = sh(script: """curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/wfapi/describe" """, returnStdout: true).trim()
                    if (!pipelineData || pipelineData == "null") { error "Failed to fetch wfapi data!" }
                    
                    // Fetch Build Metadata
                    def buildData = sh(script: """curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/api/json?tree=number,result,timestamp,estimatedDuration,executor[node,executorUtilization],artifacts[url],url" """, returnStdout: true).trim()
                    if (!buildData || buildData == "null") { error "Failed to fetch build metadata!" }

                    // Fetch Git Metadata using wfapi
                    def gitData = sh(script: """curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/wfapi/changesets" """, returnStdout: true).trim()
                    if (!gitData || gitData == "null") { gitData = '[]' }  // Handle empty changesets

                    // Extract commit details
                    def commits = []
                    def repoUrl = ""
                    def parsedGitData = readJSON(text: gitData)

                    if (parsedGitData && parsedGitData.size() > 0) {
                        repoUrl = parsedGitData[0]?.url ?: ""
                        parsedGitData.each { changeset ->
                            changeset.commits.each { commitEntry ->
                                commits << [
                                    commitId: commitEntry.commitId,
                                    commitUrl: repoUrl ? "${repoUrl}/commit/${commitEntry.commitId}" : "",
                                    message: commitEntry.msg,
                                    timestamp: commitEntry.timestamp,
                                    consoleUrl: "$JENKINS_URL${commitEntry.consoleUrl}"
                                ]
                            }
                        }
                    }

                    // Create final payload
                    def payload = [
                        pipeline: readJSON(text: pipelineData),
                        build: readJSON(text: buildData),
                        git: [
                            repository: repoUrl,
                            commits: commits
                        ]
                    ]

                    echo "Formatted Payload: ${payload}"

                    // Send Data to API
                    def response = sh(script: """curl -s -o response.json -w "%{http_code}" -X POST "$API_URL" -H "Content-Type: application/json" --data '${groovy.json.JsonOutput.toJson(payload)}'""", returnStdout: true).trim()
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
