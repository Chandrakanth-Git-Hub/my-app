@Library('my-shared-lib') _

pipeline {
    agent { label 'linux-build-1' }

    triggers {
    	githubPush()           // Trigger on GitHub webhook
    	cron('H 2 * * *')      // Also run nightly at 2 AM
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                script {
                    common.info("Code checked out successfully!")
                }
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
                        echo "Running integration tests..."
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
                    sh "echo 'uploading app.zip to server...'"
                }

                script {
                    common.info("Artifact published successfully!")
                }
            }
        }
    }

    post {
        always {
            echo "Post: Always executed"
        }
        success {
            script {
                common.notify("SUCCESS")
            }
		emailext (
           		 subject: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           		 body: "Good news! The build was successful.",
           		 to: "chandubhavi123@gmail.com"
       		 )
        }
        failure {
            script {
                common.notify("FAILURE")
            }
		emailext (
           		 subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
           		 body: "Oops! The build failed. Please check Jenkins logs.",
           		 to: "chandubhavi123@gmail.com"
       		 )
        }
    }
}

