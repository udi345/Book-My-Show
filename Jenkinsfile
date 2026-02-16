pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ACCOUNT_ID = "320674390565"
        ECR_REPO = "bookmyshow"
        IMAGE_TAG = "${BUILD_NUMBER}"
        ECR_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
        CLUSTER_NAME = "bookmyshow-cluster"
        NAMESPACE_DEV = "dev"
        NAMESPACE_STAGE = "stage"
        NAMESPACE_PROD = "prod"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/<YOUR-USERNAME>/<YOUR-REPO>.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $ECR_REPO:$IMAGE_TAG .
                docker tag $ECR_REPO:$IMAGE_TAG $ECR_URI:$IMAGE_TAG
                docker tag $ECR_REPO:$IMAGE_TAG $ECR_URI:latest
                '''
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                docker push $ECR_URI:$IMAGE_TAG
                docker push $ECR_URI:latest
                '''
            }
        }

        stage('Connect to EKS') {
            steps {
                sh '''
                aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
                '''
            }
        }

        stage('Create Namespaces') {
            steps {
                sh '''
                kubectl create namespace $NAMESPACE_DEV --dry-run=client -o yaml | kubectl apply -f -
                kubectl create namespace $NAMESPACE_STAGE --dry-run=client -o yaml | kubectl apply -f -
                kubectl create namespace $NAMESPACE_PROD --dry-run=client -o yaml | kubectl apply -f -
                '''
            }
        }

        stage('Deploy to DEV') {
            steps {
                sh '''
                sed -i "s|IMAGE_PLACEHOLDER|$ECR_URI:$IMAGE_TAG|g" k8s/deployment.yaml
                kubectl apply -f k8s/ -n $NAMESPACE_DEV
                kubectl rollout status deployment/bookmyshow -n $NAMESPACE_DEV
                '''
            }
        }

        stage('Deploy to STAGE') {
            steps {
                sh '''
                kubectl apply -f k8s/ -n $NAMESPACE_STAGE
                kubectl rollout status deployment/bookmyshow -n $NAMESPACE_STAGE
                '''
            }
        }

        stage('Deploy to PROD') {
            steps {
                sh '''
                kubectl apply -f k8s/ -n $NAMESPACE_PROD
                kubectl rollout status deployment/bookmyshow -n $NAMESPACE_PROD
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment Successful 🚀"
        }
        failure {
            echo "Deployment Failed ❌"
        }
    }
}
