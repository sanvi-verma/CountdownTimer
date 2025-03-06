pipeline {
    agent any

    environment {
        JENKINS_URL = env.JENKINS_URL ?: "http://localhost:8080"
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
                    echo "Fetching Jenkins Metadata..."

                    // ✅ Fetch Build Metadata (includes Git URL, parameters, etc.)
                    def buildData = sh(script: """curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/api/json?depth=1" """, returnStdout: true).trim()
                    if (!buildData || buildData == "null") { error "Failed to fetch build metadata!" }
                    echo "Extracted Build Data: ${buildData}"
                    def buildJson = readJSON(text: buildData)

                    // ✅ Extract Repository URL dynamically from Jenkins API
                    def repoUrl = buildJson?.actions.find { it._class == "hudson.plugins.git.util.BuildData" }?.remoteUrls?.get(0) ?: "UNKNOWN"
                    echo "Extracted Repository URL: ${repoUrl}"

                    // ✅ Fetch Pipeline Metadata
                    def pipelineData = sh(script: """curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/wfapi/describe" """, returnStdout: true).trim()
                    if (!pipelineData || pipelineData == "null") { error "Failed to fetch wfapi data!" }
                    echo "Extracted Pipeline Data: ${pipelineData}"

                    // ✅ Fetch Git ChangeSets (Commit Details)
                    def changeSetsData = sh(script: """curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/wfapi/changesets" """, returnStdout: true).trim()
                    if (!changeSetsData || changeSetsData == "null" || changeSetsData == "[]") {
                        echo "No Git changes found for this build."
                        changeSetsData = "[]"
                    } else {
                        echo "Extracted Git ChangeSets: ${changeSetsData}"
                    }

                    // ✅ Parse JSON to Extract Git Commits
                    def gitData = readJSON(text: changeSetsData)
                    def commits = []

                    if (gitData.size() > 0) {
                        for (commitEntry in gitData[0].commits) {
                            def commitId = commitEntry.commitId
                            def commitUrl = repoUrl != "UNKNOWN" ? "${repoUrl}/commit/${commitId}" : "UNKNOWN"
                            def author = commitEntry.authorJenkinsId ?: "unknown"
                            def message = commitEntry.message
                            def timestamp = commitEntry.timestamp
                            def consoleUrl = "$JENKINS_URL${commitEntry.consoleUrl}"

                            commits.add([
                                commitId    : commitId,
                                commitUrl   : commitUrl,
                                author      : author,
                                message     : message,
                                timestamp   : timestamp,
                                consoleUrl  : consoleUrl
                            ])
                        }
                    }

                    // ✅ Merge All Data into a Single JSON Object
                    def payload = """{
                        "pipeline": ${pipelineData},
                        "build": ${buildData},
                        "git": {
                            "kind": "git",
                            "repoUrl": "${repoUrl}",
                            "commitCount": ${commits.size()},
                            "commits": ${groovy.json.JsonOutput.toJson(commits)}
                        }
                    }"""

                    echo "Complete Metadata: ${payload}"

                    // ✅ Send Data to API
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
