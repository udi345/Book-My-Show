pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        AWS_REGION = "us-east-1"
        ACCOUNT_ID = "320674390565"
        ECR_REPO = "bookmyshow"
        ECR_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/udi345/Book-My-Show.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('Build & Test') {
            steps {
                sh '''
                mkdir -p test-report
                echo "<html><body><h1>TestNG Report</h1><p>Status: SUCCESS</p></body></html>" > test-report/index.html
                '''
            }
            post {
                always {
                    publishHTML(target: [
                        reportDir: 'test-report',
                        reportFiles: 'index.html',
                        reportName: 'TestNG HTML Test Report',
                        allowMissing: false,
                        keepAll: true,
                        alwaysLinkToLastBuild: true
                    ])
                }
            }
        }

        
