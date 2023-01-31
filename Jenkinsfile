pipeline{
    agent any
    //Declare environment variables
    environment {
        ECR_IMAGE="049721949876.dkr.ecr.us-east-1.amazonaws.com/node_app:v${BUILD_NUMBER}"
        APP_CONTAINER_NAME="cap-project-app-container"
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
                    if [ `docker ps | wc -l` -gt 1 ]
                    then
                        docker kill `docker ps | grep ${APP_CONTAINER_NAME} | cut -d" " -f1`
                        echo "Killed existing docker container"
                    fi
                    docker container run -d -p 8080:8080 --name ${APP_CONTAINER_NAME} ${ECR_IMAGE}
                '''
            }
        }
       }
    }
}
