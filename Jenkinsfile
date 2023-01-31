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
                    #Check for existing docker container process & kill it
                    if [ `docker ps | grep ${APP_CONTAINER_NAME} | wc -l` -eq 1 ]
                    then
                        echo -n "Killing following docker container process: "
                        docker kill `docker ps | grep ${APP_CONTAINER_NAME} | cut -d" " -f1`
                        echo "Killed above docker container process."
                    fi

                    #Check for existing docker container & remove it
                    if [ `docker ps -a | grep ${APP_CONTAINER_NAME} | wc -l` -eq 1 ]
                    then
                        echo -n "Removing following docker container: "
                        docker rm `docker ps -a | grep ${APP_CONTAINER_NAME} | cut -d" " -f1`
                        echo "Removed above docker container."
                    fi
                    docker container run -d -p 8080:8080 --name ${APP_CONTAINER_NAME} ${ECR_IMAGE}
                '''
            }
        }
       }
    }
}
