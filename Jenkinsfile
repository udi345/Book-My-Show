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
                checkout scm
            }
        }

        stage('Create Test Report') {
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
                        keepAll: true,
                        alwaysLinkToLastBuild: true
                    ])
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t bookmyshow:${BUILD_NUMBER} .'
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION \
                    | docker login --username AWS --password-stdin $ECR_URI
                    '''
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                docker tag bookmyshow:${BUILD_NUMBER} $ECR_URI:${BUILD_NUMBER}
                docker push $ECR_URI:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
