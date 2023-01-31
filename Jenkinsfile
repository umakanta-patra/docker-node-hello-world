pipeline{
    agent any
    environment {
        ECR_IMAGE="049721949876.dkr.ecr.us-east-1.amazonaws.com/node_app:v${BUILD_NUMBER}"
    }
    stages{
       stage('Git Checkout') {
        steps{
            checkout scm
        }
       } 
       stage('Docker Build and Push') {
        steps{
            sh '''
                aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 049721949876.dkr.ecr.us-east-1.amazonaws.com
                docker build -t ${ECR_IMAGE} .
                docker push ${ECR_IMAGE}
            '''
        }
       }
       stage('Deploy to app host') {
        steps{
            sshagent(credentials : ['agent']) {
                sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@10.0.2.150
                    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 049721949876.dkr.ecr.us-east-1.amazonaws.com
                    docker container run -d -p 8080:8080 --name cap-project-app-container ${ECR_IMAGE}
                '''
            }
        }
       }
    }
}
