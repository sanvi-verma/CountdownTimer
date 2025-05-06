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
                    def getRawJson = { url ->
                        sh(script: "curl -s -u '$JENKINS_USERNAME:$API_TOKEN' '${url}'", returnStdout: true).trim()
                    }
   
                    def buildData = getRawJson("${env.JENKINS_URL}/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/api/json")
                    def stageData = getRawJson("${env.JENKINS_URL}/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/wfapi/describe")
   
                    def timestamp = System.currentTimeMillis().toString()
   
                   // def encryptedTimestamp = sh(script: """
                     //   echo -n '${timestamp}' | openssl enc -aes-256-cbc -base64 \\
                       // -K \$(echo -n '${SECRET_KEY}' | xxd -p | tr -d '\\n') \\
                        //-iv \$(echo -n '${IV_KEY}' | xxd -p | tr -d '\\n')
                    //""", returnStdout: true).trim()
   
                    // Parse stage data to get nodeIds (but don't keep parsed objects)
                    def stageIds = sh(
                        script: """curl -s -u '$JENKINS_USERNAME:$API_TOKEN' '${env.JENKINS_URL}/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/wfapi/describe' | jq -r '.stages[].id'""",
                        returnStdout: true
                    ).trim().split('\n')
                   
   
   
                    // Manually build node_stage_data array as a string
                    def nodeStageDataStr = stageIds.collect { nodeId ->
                        def nodeJsonRaw = sh(
                            script: """curl -s -u '$JENKINS_USERNAME:$API_TOKEN' '${env.JENKINS_URL}/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/execution/node/${nodeId}/wfapi/describe'""",
                            returnStdout: true
                        ).trim()
                        return """{"nodeId":${groovy.json.JsonOutput.toJson(nodeId)},"data":${nodeJsonRaw}}"""
                    }.join(',')
def webhookUrl = 'https://1d41-192-245-162-37.ngrok-free.app/web'
   
   
                    // Final payload as JSON string
                    def payload = """{
                        "build_data": ${buildData},
                        "node_stage_data": [${nodeStageDataStr}]
                    }""".replace("'", "'\"'\"'")
                   
                    // Calculate SHA256 checksum of the payload
                    def checksum = sh(script: "echo -n '${payload}' | sha256sum | awk '{print \$1}'", returnStdout: true).trim()
                   
                    // Send the payload with checksum and encrypted timestamp
                    sh """
                        curl -X POST '${webhookUrl}' \\
                        -H "Content-Type: application/json" \\
                        -H "X-Encrypted-Timestamp: ${encryptedTimestamp}" \\
                        -H "X-Checksum: ${checksum}" \\
                        -d '${payload}'
                    """
                }
            }
        }
    }

}
