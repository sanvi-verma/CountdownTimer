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
                    // Fetch and parse pipeline metadata
                    def pipelineRaw = sh(script: """curl -s -X GET "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/wfapi/describe" """, returnStdout: true).trim()
                    def pipelineJson = readJSON text: pipelineRaw

                    // Fetch and parse build metadata
                    def buildRaw = sh(script: """curl -s -X GET "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/api/json?depth=1" """, returnStdout: true).trim()
                    def buildJson = readJSON text: buildRaw

                    // Extract Git Data
                    def gitBranch = sh(script: 'echo $GIT_BRANCH', returnStdout: true).trim()
                    def gitCommit = sh(script: 'echo $GIT_COMMIT', returnStdout: true).trim()

                    // **Create a simple JSON structure to avoid circular references**
                    def formattedData = [
                        pipeline: [
                            id: pipelineJson.id ?: buildJson.number,
                            name: JOB_NAME,
                            status: pipelineJson.status ?: "IN_PROGRESS",
                            startTime: pipelineJson.startTimeMillis ?: buildJson.timestamp,
                            endTime: pipelineJson.endTimeMillis ?: 0,
                            duration: pipelineJson.durationMillis ?: 0,
                            url: "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/",
                            stages: pipelineJson.stages.collect { stage ->
                                [
                                    stage: stage.name,
                                    status: stage.status,
                                    startTime: stage.startTimeMillis,
                                    endTime: stage.endTimeMillis ?: 0,
                                    duration: stage.durationMillis
                                ]
                            }
                        ],
                        build: [
                            buildNumber: buildJson.number,
                            status: buildJson.result ?: "IN_PROGRESS",
                            triggeredBy: [
                                userId: buildJson.actions.find { it.causes }?.causes[0]?.userId ?: "unknown",
                                userName: buildJson.actions.find { it.causes }?.causes[0]?.userName ?: "unknown"
                            ],
                            timestamp: buildJson.timestamp,
                            duration: buildJson.duration ?: 0,
                            estimatedDuration: buildJson.estimatedDuration,
                            executor: [
                                node: buildJson.builtOn ?: "built-in",
                                executorUtilization: 1
                            ],
                            artifactsUrl: "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/artifact",
                            displayUrl: "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/pipeline-graph",
                            testsUrl: "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/testReport"
                        ],
                        git: [
                            message: gitBranch ? "Git data available" : "No git data available"
                        ]
                    ]

                    // Convert structured data to JSON safely
                    def jsonPayload = groovy.json.JsonOutput.toJson(formattedData)

                    // Send data to API
                    sh """curl -X POST "$API_URL" \
                        -H "Content-Type: application/json" \
                        -d '${jsonPayload}'"""
                }
            }
        }
    }
}
