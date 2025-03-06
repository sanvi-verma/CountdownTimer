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
            try {
                // Fetch pipeline metadata
                def pipelineDataRaw = sh(script: """curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/wfapi/describe" """, returnStdout: true).trim()
                def pipelineData = pipelineDataRaw ? readJSON(text: pipelineDataRaw) : [:]

                // Fetch build details
                def buildDataRaw = sh(script: """curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/api/json?depth=1" """, returnStdout: true).trim()
                def buildData = buildDataRaw ? readJSON(text: buildDataRaw) : [:]

                // Convert data to pure Map objects to avoid serialization errors
                def jsonPayload = [
                    pipeline: [
                        id       : pipelineData?.id?.toString() ?: "Unknown",
                        name     : pipelineData?.name ?: "Unknown",
                        status   : pipelineData?.status ?: "UNKNOWN",
                        startTime: pipelineData?.startTimeMillis ?: 0,
                        endTime  : pipelineData?.endTimeMillis ?: 0,
                        duration : pipelineData?.durationMillis ?: 0,
                        url      : pipelineData?.url ?: "Unknown",
                        stages   : pipelineData?.stages?.collect { stage ->
                            [
                                stage    : stage?.name ?: "Unknown",
                                status   : stage?.status ?: "UNKNOWN",
                                startTime: stage?.startTimeMillis ?: 0,
                                duration : stage?.durationMillis ?: 0
                            ]
                        } ?: []
                    ],
                    build   : [
                        buildNumber       : buildData?.number?.toString() ?: "Unknown",
                        status           : buildData?.result ?: "IN_PROGRESS",
                        timestamp        : buildData?.timestamp ?: 0,
                        duration         : buildData?.duration ?: 0,
                        estimatedDuration: buildData?.estimatedDuration ?: 0,
                        executor         : [
                            node                : buildData?.builtOn ?: "built-in"
                        ],
                        artifactsUrl     : buildData?.artifacts?.size() > 0 ? buildData.artifacts[0].url : "Unknown",
                        displayUrl       : buildData?.url ?: "Unknown",
                        testsUrl         : buildData?.url ? "${buildData.url}testReport" : "Unknown"
                    ],
                    git     : [
                        message: "No git data available"
                    ]
                ]

                // Safe JSON conversion
                def jsonString = groovy.json.JsonOutput.toJson(jsonPayload)
                def safeJsonString = groovy.json.JsonOutput.prettyPrint(jsonString)

                // Print & Send Data
                echo "Complete Metadata: ${safeJsonString}"
                sh """curl -X POST "$API_URL" \
                    -H "Content-Type: application/json" \
                    -d '${jsonString}'"""

            } catch (Exception e) {
                echo "❌ ERROR: ${e.getMessage()}"
            }
        }
    }
}

    }
}
