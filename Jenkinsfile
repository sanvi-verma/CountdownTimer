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
                    def pipelineDataRaw = sh(script: """curl -s -X GET "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/wfapi/describe" """, returnStdout: true).trim()
                    def pipelineData = readJSON(text: pipelineDataRaw)

                    def buildDataRaw = sh(script: """curl -s -X GET "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/api/json?depth=1" """, returnStdout: true).trim()
                    def buildData = readJSON(text: buildDataRaw)

                    def gitBranch = sh(script: 'echo $GIT_BRANCH', returnStdout: true).trim()
                    def gitCommit = sh(script: 'echo $GIT_COMMIT', returnStdout: true).trim()

                    def formattedPipeline = [
                        id        : pipelineData?.id ?: "Unknown",
                        name      : pipelineData?.name ?: "Unknown",
                        status    : pipelineData?.status ?: "UNKNOWN",
                        startTime : pipelineData?.startTimeMillis ?: 0,
                        endTime   : pipelineData?.endTimeMillis ?: 0,
                        duration  : pipelineData?.durationMillis ?: 0,
                        url       : pipelineData?.url ?: "Unknown",
                        stages    : pipelineData?.stages?.collect { stage ->
                            [
                                stage    : stage.name ?: "Unknown",
                                status   : stage.status ?: "UNKNOWN",
                                startTime: stage.startTimeMillis ?: 0,
                                duration : stage.durationMillis ?: 0
                            ]
                        } ?: []
                    ]

                    def formattedBuild = [
                        buildNumber        : buildData?.number ?: "Unknown",
                        status            : buildData?.result ?: "UNKNOWN",
                        triggeredBy       : [
                            userId  : buildData?.actions?.find { it.causes }?.causes?.first()?.userId ?: "unknown",
                            userName: buildData?.actions?.find { it.causes }?.causes?.first()?.userName ?: "unknown"
                        ],
                        timestamp         : buildData?.timestamp ?: 0,
                        estimatedDuration : buildData?.estimatedDuration ?: 0,
                        executor         : [
                            node                : buildData?.builtOn ?: "built-in",
                            executorUtilization : buildData?.executor?.executorUtilization ?: 1
                        ],
                        artifactsUrl     : buildData?.artifacts?.size() > 0 ? buildData.artifacts[0].url : "Unknown",
                        displayUrl       : buildData?.url ?: "Unknown",
                        testsUrl         : buildData?.url ? "${buildData.url}testReport" : "Unknown"
                    ]

                    def formattedGit = gitCommit ? [
                        branch : gitBranch,
                        commit : gitCommit
                    ] : [message: "No git data available"]

                    def jsonPayload = groovy.json.JsonOutput.toJson([
                        pipeline: formattedPipeline as Map,
                        build   : formattedBuild as Map,
                        git     : formattedGit as Map
                    ])

                    echo "Complete Metadata: ${groovy.json.JsonOutput.prettyPrint(jsonPayload)}"

                    sh """curl -X POST "$API_URL" \
                        -H "Content-Type: application/json" \
                        -d '${jsonPayload}'"""
                }
            }
        }
    }
}
