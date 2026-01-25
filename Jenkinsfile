// Trigger build

pipeline {
    agent any

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/<your-username>/Book-My-Show.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t bookmyshow:${BUILD_NUMBER} .'
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh 'kubectl apply -f deployment.yaml -n staging'
            }
        }

        stage('Deploy to Production') {
            steps {
                sh 'kubectl apply -f deployment.yaml -n production'
            }
        }
    }
}
