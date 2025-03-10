
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
                def webhookUrl = "https://a2a9-2409-4081-2c88-1f9f-1a7-28d7-fcc-811a.ngrok-free.app/webhook"

                def wfapiResponse = sh(script: "curl -s ${apiUrl}", returnStdout: true).trim()
                def jsonApiResponse = sh(script: "curl -s ${jsonApiUrl}", returnStdout: true).trim()

                def payload = """
                {
                    "api_json": ${jsonApiResponse},
                    "wfapi_describe": ${wfapiResponse}
                }
                """

                sh """
                curl -X POST "${webhookUrl}" \
                     -H "Content-Type: application/json" \
                     -H "X-Encrypted-Timestamp: $(date +%s | rev)" \
                     -H "X-Payload-Checksum: $(echo -n '${payload}' | sha256sum | awk '{print \$1}')" \
                     -d '${payload}'
                """
            }
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
