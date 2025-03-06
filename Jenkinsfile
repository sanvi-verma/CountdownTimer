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

                        // Extract pipeline details safely
                        def formattedPipeline = [
                            id       : pipelineData?.id ?: "Unknown",
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
                        ]

                        // Extract build details safely
                        def formattedBuild = [
                            buildNumber       : buildData?.number?.toString() ?: "Unknown",
                            status           : buildData?.result ?: "IN_PROGRESS",
                            triggeredBy      : [
                                userId  : buildData?.actions?.find { it.causes }?.causes?.first()?.userId ?: "unknown",
                                userName: buildData?.actions?.find { it.causes }?.causes?.first()?.userName ?: "unknown"
                            ],
                            timestamp        : buildData?.timestamp ?: 0,
                            duration         : buildData?.duration ?: 0,
                            estimatedDuration: buildData?.estimatedDuration ?: 0,
                            executor         : [
                                node                : buildData?.builtOn ?: "built-in",
                                executorUtilization : buildData?.executor?.executorUtilization ?: 1
                            ],
                            artifactsUrl     : buildData?.artifacts?.size() > 0 ? buildData.artifacts[0].url : "Unknown",
                            displayUrl       : buildData?.url ?: "Unknown",
                            testsUrl         : buildData?.url ? "${buildData.url}testReport" : "Unknown"
                        ]

                        // Construct final JSON payload safely
                        def jsonPayload = [
                            pipeline: formattedPipeline,
                            build   : formattedBuild,
                            git     : [
                                message: "No git data available"
                            ]
                        ]

                        // Convert to JSON safely (handling special cases)
                        def jsonString = groovy.json.JsonOutput.toJson(jsonPayload)
                        def safeJsonString = groovy.json.JsonOutput.prettyPrint(jsonString)

                        // Print and send the payload
                        echo "Complete Metadata: ${safeJsonString}"

                        sh """curl -X POST "$API_URL" \
                            -H "Content-Type: application/json" \
                            -d '${jsonString}'"""
                        
                    } catch (Exception e) {
                        echo "Error while processing JSON: ${e}"
                    }
                }
            }
        }
    }
}
