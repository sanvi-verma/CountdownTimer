pipeline {
    agent any
environment
    {
        WEBHOOK_URL = 'https://d507-192-245-162-37.ngrok-free.app/web'
    }
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
                    def stageDescribe = getRawJson("${env.JENKINS_URL}/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/wfapi/describe")

                    // ✅ Parse JSON with readJSON
                    writeFile file: 'stageDescribe.json', text: stageDescribe
                    def parsedDescribe = readJSON file: 'stageDescribe.json'

                    def nodeStageDataStr = parsedDescribe.stages.collect { stage ->
                        def nodeId = stage.id
                        def nodeData = getRawJson("${env.JENKINS_URL}/job/${env.JOB_NAME}/${env.BUILD_NUMBER}/execution/node/${nodeId}/wfapi/describe")
                        return """{"nodeId":${groovy.json.JsonOutput.toJson(nodeId)},"data":${nodeData}}"""
                    }.join(',')

                    def payloadStr = """{"build_data": ${buildData}, "node_stage_data": [${nodeStageDataStr}]}"""
                    writeFile file: 'payload.json', text: payloadStr

                    def checksum = sh(script: "sha256sum payload.json | awk '{print \$1}'", returnStdout: true).trim()

                    // def timestamp = System.currentTimeMillis().toString()
                    // def encryptedTimestamp = sh(script: """
                    //     echo -n '${timestamp}' | openssl enc -aes-256-cbc -base64 \\
                    //     -K \$(echo -n '${SECRET_KEY}' | xxd -p | tr -d '\\n') \\
                    //     -iv \$(echo -n '${IV_KEY}' | xxd -p | tr -d '\\n')
                    // """, returnStdout: true).trim()

                    sh """
                        curl -X POST '${WEBHOOK_URL}' \\
                        -H "Content-Type: application/json" \\
                        -H "X-Checksum: ${checksum}" \\
                        --data-binary @payload.json
                    """
                }
            }
        }
    }

}

