
pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
            }
        }

        stage('Set Up Environment') {
            steps {
                echo 'Setting up environment...'
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
    }

    post {
                success {
            script {
                def jenkinsUrl = env.JENKINS_URL ?: 'http://localhost:8080'
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def apiUrl = "${jenkinsUrl}/job/${jobName}/${buildNumber}/wfapi/describe"
                def jsonApiUrl = "${jenkinsUrl}/job/${jobName}/${buildNumber}/api/json"
                def webhookUrl = "https://6451-2409-40c2-115f-95f5-45da-9eeb-7cc7-5618.ngrok-free.app/jenkins-metadata"

                def wfapiResponse = sh(script: "curl -s ${apiUrl}", returnStdout: true).trim()
                def jsonApiResponse = sh(script: "curl -s ${jsonApiUrl}", returnStdout: true).trim()

                // Filter out the FlowGraphAction before sending
                def parsedJson = readJSON text: jsonApiResponse
                parsedJson.actions = parsedJson.actions.findAll { it._class != "org.jenkinsci.plugins.workflow.job.views.FlowGraphAction" }

                def cleanedJsonApiResponse = writeJSON returnText: true, json: parsedJson

                def payload = """
                {
                    "api_json": ${cleanedJsonApiResponse},
                    "wfapi_describe": ${wfapiResponse}
                }
                """

                sh """
                curl -X POST "${webhookUrl}" \
                     -H "Content-Type: application/json" \
                     -H "X-Encrypted-Timestamp: $(date +%s)" \
                     -H "X-Payload-Checksum: $(echo -n '${payload}' | sha256sum | awk '{print $1}')" \
                     -d '${payload}'
                """
            }
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
