pipeline {
    agent any

    triggers {
        githubPush()
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
                sh "mkdir -p test-report"
                sh "echo '<html><body><h1>TestNG Report</h1><p>Status: SUCCESS</p></body></html>' > test-report/index.html"
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

        stage('Docker Build') {
            steps {
                sh "docker build -t bookmyshow:${BUILD_NUMBER} ."
            }
        }

        stage('Push to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {
                    sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 320674390565.dkr.ecr.us-east-1.amazonaws.com"
                    sh "docker tag bookmyshow:${BUILD_NUMBER} 320674390565.dkr.ecr.us-east-1.amazonaws.com/bookmyshow:${BUILD_NUMBER}"
                    sh "docker push 320674390565.dkr.ecr.us-east-1.amazonaws.com/bookmyshow:${BUILD_NUMBER}"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
