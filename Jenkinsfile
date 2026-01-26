pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "bookmyshow"
        ECR_URI = "320674390565.dkr.ecr.us-east-1.amazonaws.com"
        IMAGE_TAG = "latest"
        CLUSTER_NAME = "bookmyshow-cluster"
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
                sh '''
                  cd bookmyshow-app
                  npm install
                  npm test || true
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                  docker build -t $ECR_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('AWS ECR Login') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh '''
                      aws ecr get-login-password --region $AWS_REGION \
                      | docker login --username AWS --password-stdin $ECR_URI
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                  docker tag $ECR_REPO:$IMAGE_TAG $ECR_URI/$ECR_REPO:$IMAGE_TAG
                  docker push $ECR_URI/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to EKS (DEV)') {
            steps {
                sh '''
                  aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME
                  kubectl apply -f deployment.yml -n dev
                  kubectl apply -f service.yml -n dev
                '''
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline completed successfully"
        }
        failure {
            echo "CI/CD Pipeline failed"
        }
    }
}

