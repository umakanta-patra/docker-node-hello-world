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
                #Authenticate docker client to ecr
                aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 049721949876.dkr.ecr.us-east-1.amazonaws.com
                
                #Build app docker image & push to ecr
                docker build -t ${ECR_IMAGE} .
                docker push ${ECR_IMAGE}
            '''
        }
       }
       stage('Deploy to app host') {
        steps{
            sshagent(credentials : ['agent']) {
                sh '''
                    #ssh to app machine and authenticate docker client to ecr
                    ssh -o StrictHostKeyChecking=no ubuntu@10.0.2.150
                    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 049721949876.dkr.ecr.us-east-1.amazonaws.com
                    
                    #Check for existing docker container & kill/remove it
                    if [ `docker ps | grep ${APP_CONTAINER_NAME} | wc -l` -eq 1 ]
                    then
                        container_id=`docker ps | grep ${APP_CONTAINER_NAME} | cut -d" " -f1`
                        echo "Killing/Removing following docker container: ${container_id}"
                        docker kill ${container_id}
                        docker rm ${container_id}
                        echo "Killed/Removed above docker container process."
                    fi

                    #Create and run app image on app machine
                    docker container run -d -p 8080:8080 --name ${APP_CONTAINER_NAME} ${ECR_IMAGE}
                '''
            }
        }
       }
    }
}
