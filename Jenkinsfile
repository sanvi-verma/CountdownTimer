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
                def webhookUrl = "https://d3a9-2402-e280-3e1d-bce-9044-f6-e56a-7441.ngrok-free.app/jenkins-metadata"

                def wfapiResponse = sh(script: "curl -s ${apiUrl}", returnStdout: true).trim()
                def jsonApiResponse = sh(script: "curl -s ${jsonApiUrl}", returnStdout: true).trim()

                // Convert to valid JSON string by escaping quotes
                def payload = """
                {
                    "api_json": ${jsonApiResponse},
                    "wfapi_describe": ${wfapiResponse}
                }
                """.stripIndent()

                // Write payload to file to avoid issues with shell escaping
                writeFile file: 'payload.json', text: payload

                // Debugging: Print the payload
                echo "Payload being sent: ${payload}"

                sh '''
                curl -X POST "https://d3a9-2402-e280-3e1d-bce-9044-f6-e56a-7441.ngrok-free.app/jenkins-metadata" \
                     -H "Content-Type: application/json" \
                     -H "X-Encrypted-Timestamp: $(date +%s)" \
                     -H "X-Payload-Checksum: $(cat payload.json | sha256sum | awk '{print $1}')" \
                     --data-binary @payload.json
                '''
            }
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
