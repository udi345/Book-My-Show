pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        AWS_ACCOUNT_ID = "320674390565"
        AWS_REGION     = "us-east-1"
        ECR_REPO       = "bookmyshow"
        IMAGE_TAG      = "${BUILD_NUMBER}"
        ECR_URI        = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/udi345/Book-My-Show.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('Build & Test') {
            steps {
                echo "Build & test stage"
            }
        }
    }

    post {
        always {
            echo "Pipeline completed"
        }
    }
}
