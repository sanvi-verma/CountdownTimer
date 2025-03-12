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
                def webhookUrl = "https://29e9-2409-40c2-115f-95f5-1546-899-3e20-72b4.ngrok-free.app/jenkins-metadata".trim()

                // Ensure webhook URL is set
                if (!webhookUrl) {
                    error("Webhook URL is missing or invalid")
                }

                def wfapiResponse = sh(script: "curl -s '${apiUrl}'", returnStdout: true).trim()
                def jsonApiResponse = sh(script: "curl -s '${jsonApiUrl}'", returnStdout: true).trim()

                def parsedJson = readJSON text: jsonApiResponse
                parsedJson.actions = parsedJson.actions.findAll { it._class != "org.jenkinsci.plugins.workflow.job.views.FlowGraphAction" }
                def cleanedJsonApiResponse = writeJSON returnText: true, json: parsedJson

                def payload = """{
                    "api_json": ${cleanedJsonApiResponse},
                    "wfapi_describe": ${wfapiResponse}
                }""".stripIndent()

                // Save payload to a temporary file
                writeFile file: 'payload.json', text: payload

                sh '''
                curl -X POST '${webhookUrl}' \
                     -H "Content-Type: application/json" \
                     -H "X-Encrypted-Timestamp: $(date +%s)" \
                     -H "X-Payload-Checksum: $(sha256sum payload.json | awk '{print $1}')" \
                     --data-binary @payload.json
                '''
            }
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
