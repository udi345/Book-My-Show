pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ACCOUNT_ID = "320674390565"
        ECR_REPO = "bookmyshow"
        IMAGE_TAG = "${BUILD_NUMBER}"
        ECR_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
        CLUSTER_NAME = "bookmyshow-cluster"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/udi345/Book-My-Show.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t bookmyshow .'
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

        stage('Tag & Push Image') {
            steps {
                sh '''
                docker tag bookmyshow:latest $ECR_URI:$IMAGE_TAG
                docker push $ECR_URI:$IMAGE_TAG
                '''
            }
        }

        stage('Update Kubernetes Deployment File') {
            steps {
                sh '''
                sed -i "s|REPLACE_IMAGE|$ECR_URI:$IMAGE_TAG|g" deployment.yml
                '''
            }
        }

        stage('Connect to EKS') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {
                    sh '''
                    aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f deployment.yml
                kubectl apply -f service.yml
                '''
            }
        }

        stage('Verify & Rollback Deployment') {
            steps {
                script {
                    def status = sh(script: "kubectl rollout status deployment/bms-app --timeout=120s", returnStatus: true)

                    if (status != 0) {
                        sh '''
                        echo "Deployment failed. Rolling back..."
                        kubectl rollout undo deployment/bms-app
                        '''
                        error("Deployment Failed → Rollback executed")
                    } else {
                        echo "Deployment successful"
                    }
                }
            }
        }
    }
}
