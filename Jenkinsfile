pipeline {
    agent any

    environment {
        DB_URL = "jdbc:mysql://host.docker.internal:3306/jenkins_db"
        DB_USER = "jenkins"
        DB_PASSWORD = "yourpassword"
    }

    stages {
        stage('Start Build') {
            steps {
                script {
                    def startTime = new Date().format("yyyy-MM-dd HH:mm:ss")
                    env.START_TIME = startTime
                    echo "Build started at ${startTime}"
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    logStep("Clone Repository") {
                        echo "Cloning repository..."
                    }
                }
            }
        }

        stage('Set Up Environment') {
            steps {
                script {
                    logStep("Set Up Environment") {
                        echo "Setting up environment..."
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    logStep("Build") {
                        echo "Building..."
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    logStep("Test") {
                        echo "Running tests..."
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    logStep("Deploy") {
                        echo "Deploying..."
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                def endTime = new Date().format("yyyy-MM-dd HH:mm:ss")
                def duration = System.currentTimeMillis() - currentBuild.getStartTimeInMillis()

                def buildMetadata = [
                    job_name    : env.JOB_NAME,
                    build_number: env.BUILD_NUMBER,
                    triggered_by: env.BUILD_USER_ID ?: 'Unknown User',
                    start_time  : env.START_TIME,
                    end_time    : endTime,
                    duration    : duration,
                    git_branch  : env.GIT_BRANCH ?: 'Unknown'
                ]

                storeBuildMetadata(buildMetadata)
            }
        }
    }
}

def logStep(stepName, Closure body) {
    def startTime = System.currentTimeMillis()
    def status = 'SUCCESS'
    def errorMsg = ''

    try {
        body()
    } catch (Exception e) {
        status = 'FAILED'
        errorMsg = e.getMessage()
        throw e
    } finally {
        def endTime = System.currentTimeMillis()
        def duration = endTime - startTime
        def timestamp = new Date().format("yyyy-MM-dd HH:mm:ss")

        def stepData = [
            job_name    : env.JOB_NAME,
            build_number: env.BUILD_NUMBER,
            step_name   : stepName,
            status      : status,
            start_time  : timestamp,
            end_time    : new Date().format("yyyy-MM-dd HH:mm:ss"),
            duration    : duration,
            triggered_by: env.BUILD_USER_ID ?: 'Unknown User',
            git_branch  : env.GIT_BRANCH ?: 'Unknown',
            node_name   : env.NODE_NAME
        ]

        storeStepData(stepData)
    }
}

def storeStepData(def data) {
    def query = """
        INSERT INTO step_execution (job_name, build_number, step_name, status, start_time, end_time, duration, triggered_by, git_branch, node_name)
        VALUES ('${data.job_name}', ${data.build_number}, '${data.step_name}', '${data.status}', '${data.start_time}', '${data.end_time}', ${data.duration}, '${data.triggered_by}', '${data.git_branch}', '${data.node_name}')
    """

    executeSQL(query)
}

def storeBuildMetadata(def data) {
    def query = """
        INSERT INTO build_metadata (job_name, build_number, triggered_by, start_time, end_time, duration, git_branch)
        VALUES ('${data.job_name}', ${data.build_number}, '${data.triggered_by}', '${data.start_time}', '${data.end_time}', ${data.duration}, '${data.git_branch}')
    """

    executeSQL(query)
}

def executeSQL(String query) {
    withCredentials([usernamePassword(credentialsId: 'mysql-credentials', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASSWORD')]) {
        def dbUrl = env.DB_URL
        def dbUser = env.DB_USER
        def dbPassword = env.DB_PASSWORD

        sh """
            mysql --user=${dbUser} --password=${dbPassword} --execute="${query}" --database=jenkins_db --host=host.docker.internal
        """
    }
}
