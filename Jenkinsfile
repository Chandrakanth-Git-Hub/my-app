@Library('my-shared-lib') _

pipeline {
    agent any

    triggers {
        // Webhook triggers automatically when push happens
        // (Enable in Jenkins job: GitHub hook trigger)
        
        // Scheduled trigger – runs every night 2 AM
        cron('0 2 * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                common.info("Code checked out successfully!")
            }
        }

        stage('Build') {
            steps {
                echo "Building application..."
                sh "echo 'print(\"Hello World\")' > build_output.txt"
                stash name: 'buildFiles', includes: 'build_output.txt'
            }
        }

        stage('Tests (Parallel)') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        unstash 'buildFiles'
                        echo "Running unit tests..."
                        sh "echo 'Unit Test Passed' > unit_result.txt"
                        archiveArtifacts artifacts: 'unit_result.txt'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        unstash 'buildFiles'
                        echo 'Running integration tests...'
                        sh "echo 'Integration Test Passed' > integ_result.txt"
                        archiveArtifacts artifacts: 'integ_result.txt'
                    }
                }
            }
        }

        stage('Package') {
            steps {
                echo "Packaging artifact..."
                sh "zip app.zip build_output.txt unit_result.txt integ_result.txt"
                stash name: 'artifact', includes: 'app.zip'
                archiveArtifacts artifacts: 'app.zip'
            }
        }

        stage('Publish Artifact') {
            steps {
                unstash 'artifact'
                withCredentials([usernamePassword(
                    credentialsId: 'artifact-cred',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    echo "Publishing using USER=$USER"
                    sh "echo 'uploading app.zip to server...' "
                }
                common.info("Artifact published successfully!")
            }
        }
    }

    post {
        always {
            echo "Post: Always executed"
        }
        success {
            common.notify("SUCCESS")
        }
        failure {
            common.notify("FAILURE")
        }
    }
}

