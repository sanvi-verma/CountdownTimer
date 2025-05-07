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
        always {
            script {
                withCredentials([
                    string(credentialsId: 'jenkins-username', variable: 'JENKINS_USERNAME'),
                    string(credentialsId: 'api-token', variable: 'API_TOKEN'),
                    string(credentialsId: 'secret-key', variable: 'SECRET_KEY'),
                    string(credentialsId: 'iv-key', variable: 'IV_KEY')
                ]) {
                    def API_URL_1 = "${env.JENKINS_URL}/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/api/json"
                    def API_URL_2 = "${env.JENKINS_URL}/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/wfapi/describe"
                    def WEBHOOK_URL = "https://cb4f-192-245-162-37.ngrok-free.app/webhook"

                    // Fetch Jenkins API responses
                    def buildData = sh(script: "curl -s -u '${JENKINS_USERNAME}:${API_TOKEN}' '${API_URL_1}'", returnStdout: true).trim()
                    def stageData = sh(script: "curl -s -u '${JENKINS_USERNAME}:${API_TOKEN}' '${API_URL_2}'", returnStdout: true).trim()
                   
                   
                    // Compute Checksum
                    def checksum_build = sh(script: "echo -n '${buildData}' | sha256sum | awk '{print \$1}'", returnStdout: true).trim()
                    def checksum_stage = sh(script: "echo -n '${stageData}' | sha256sum | awk '{print \$1}'", returnStdout: true).trim()
                   
                    //Make the payload
                    def payload = [
                        build_data: buildData,
                        stage_data: stageData
                    ]
                    def jsonPayload = groovy.json.JsonOutput.toJson(payload)
                   
                    // Encrypt timestamp with AES-256-CBC
                    // def timestamp = System.currentTimeMillis().toString()
                    // def encryptedTimestamp = sh(script: """
                    //     echo -n '${timestamp}' | openssl enc -aes-256-cbc -base64 \\
                    //     -K \$(echo -n '${SECRET_KEY}' | xxd -p | tr -d '\\n') \\
                    //     -iv \$(echo -n '${IV_KEY}' | xxd -p | tr -d '\\n')
                    // """, returnStdout: true).trim()


                    // Send the payload with checksum as a header
                    sh """
                        curl -X POST '${WEBHOOK_URL}' \\
                        -H "Content-Type: application/json" \\
                        -H "X-Checksum-Build: ${checksum_build}" \\
                        -H "X-Checksum-Stage: ${checksum_stage}" \\
                        -d '${jsonPayload}'
                    """
                }
            }
        }
    }

}

