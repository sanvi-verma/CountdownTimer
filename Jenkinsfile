import groovy.json.JsonOutput
import javax.crypto.Cipher
import javax.crypto.spec.SecretKeySpec
import java.util.Base64

pipeline {
    agent any

    environment {
        API_URL = ' https://dcd0-2402-e280-3e1d-bce-75de-6ae9-5635-1a8b.ngrok-free.app/jenkins-metadata'
        API_KEY = 'qteyew2537e3ygdhusdhd833' 
        ENCRYPTION_KEY = 'mySecretKey12345'  
    }

    stages {
        stage('Initialize Metadata') {
            steps {
                script {
                    METADATA = []  
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    try {
                        git branch: 'main', url: 'https://github.com/sanvi-verma/CountdownTimer.git'
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Clone Repository", "SUCCESS", startTime, endTime, "Repository cloned successfully.", METADATA)
                    } catch (Exception e) {
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Clone Repository", "FAILURE", startTime, endTime, e.toString(), METADATA)
                        error("Stage failed: ${e}")
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    try {
                        echo 'Building the project...'
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Build", "SUCCESS", startTime, endTime, "Build completed.", METADATA)
                    } catch (Exception e) {
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Build", "FAILURE", startTime, endTime, e.toString(), METADATA)
                        error("Stage failed: ${e}")
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    try {
                        echo 'Running tests...'
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Test", "SUCCESS", startTime, endTime, "Tests executed successfully.", METADATA)
                    } catch (Exception e) {
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Test", "FAILURE", startTime, endTime, e.toString(), METADATA)
                        error("Stage failed: ${e}")
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def startTime = System.currentTimeMillis()
                    try {
                        echo 'Deploying the application...'
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Deploy", "SUCCESS", startTime, endTime, "Deployment completed successfully.", METADATA)
                    } catch (Exception e) {
                        def endTime = System.currentTimeMillis()
                        appendStageMetadata("Deploy", "FAILURE", startTime, endTime, e.toString(), METADATA)
                        error("Stage failed: ${e}")
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                sendFinalMetadata(METADATA)
            }
        }
    }
}

// Function to Append Stage Metadata
def appendStageMetadata(stageName, status, startTime, endTime, consoleLog, metadataList) {
    def duration = endTime - startTime
    def stepData = [
        stage: stageName,
        status: status,
        startTime: startTime,
        endTime: endTime,
        duration: duration,
        consoleLog: consoleLog
    ]

    metadataList.add(stepData)  
}


def encryptMetadata(metadataJson, secretKey) {
    def key = new SecretKeySpec(secretKey.getBytes(), "AES")
    def cipher = Cipher.getInstance("AES")
    cipher.init(Cipher.ENCRYPT_MODE, key)
    def encryptedBytes = cipher.doFinal(metadataJson.getBytes("UTF-8"))
    return Base64.encoder.encodeToString(encryptedBytes)
}

def sendFinalMetadata(metadataList) {
    def finalMetadata = [
        metadata: [
            buildNumber: env.BUILD_NUMBER,
            jobName: env.JOB_NAME,
            nodeName: env.NODE_NAME,
            executorNumber: env.EXECUTOR_NUMBER ?: 'N/A',
            buildUrl: env.BUILD_URL
        ],
        steps: metadataList
    ]

    def jsonString = JsonOutput.toJson(finalMetadata)  
    def encryptedData = encryptMetadata(jsonString, env.ENCRYPTION_KEY) 

    sh """curl -X POST ${env.API_URL} \
        -H "Content-Type: application/json" \
        -H "x-api-key: ${env.API_KEY}" \
        -d '{ "data": "${encryptedData}" }'""" 
}
