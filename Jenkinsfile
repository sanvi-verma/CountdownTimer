import groovy.json.JsonOutput
import javax.crypto.Cipher
import javax.crypto.spec.SecretKeySpec
import java.util.Base64

pipeline {
    agent any

    environment {
        JENKINS_URL = "http://localhost:8080"
        API_URL = "https://2cf9-2402-e280-3e1d-bce-2584-894f-4e39-6c7c.ngrok-free.app/jenkins-metadata"
        API_KEY = 'qteyew2537e3ygdhusdhd833' 
        ENCRYPTION_KEY = "mySecretKey12345"
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
            steps{
            echo "Cloning repo"
            }
        }

        stage('Build') {
            steps {
                echo "Building"
            }
        }

        stage('Test') {
            steps {
                echo "Testing"
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying"
            }
        }

        stage('Fetch Real-Time Data') {
            steps {
                script {
                    def jobName = env.JOB_NAME ?: "unknown-job"
                    def buildNumber = env.BUILD_NUMBER ?: "unknown-build"

                    def response = sh(script: """curl -s -X GET "$JENKINS_URL/job/$jobName/$buildNumber/wfapi/describe" """, returnStdout: true).trim()
                    echo "Real-time Pipeline Data: ${response}"

                    sh """curl -X POST "$API_URL" \
                        -H "Content-Type: application/json" \
                        -d '{ "data": ${response} }'"""
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

// Function to Send Metadata to API
def sendFinalMetadata(metadataList) {
    def finalMetadata = [
        metadata: [
            buildNumber: env.BUILD_NUMBER ?: "unknown-build",
            jobName: env.JOB_NAME ?: "unknown-job",
            nodeName: env.NODE_NAME ?: "unknown-node",
            executorNumber: env.EXECUTOR_NUMBER ?: 'N/A',
            buildUrl: env.BUILD_URL ?: "unknown-url"
        ],
        steps: metadataList
    ]

    def jsonString = JsonOutput.toJson(finalMetadata)

    sh """curl -X POST ${env.API_URL} \
        -H "Content-Type: application/json" \
        -d '{ "data": ${jsonString} }'"""
}
