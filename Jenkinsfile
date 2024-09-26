pipeline {
    agent any
    environment {
        DOCKER_REGISTRY_NAME = "966508915346.dkr.ecr.us-east-1.amazonaws.com"
        DOCKER_IMAGE_NAME = "crwdlab-cicd"
     // DOCKERHUB_CREDENTIALS= credentials('dockerhubcredentials')

        }
    
stages {
    stage('Build Docker Image') {
            when {
                branch 'main'
            }
        steps {
                echo 'Building docker image'
                sh "docker build -t ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ."
            }
        }

    stage('Push Docker Image to ECR') {
        when {
            branch 'main'
        }
        environment {
        AWS_ACCESS_KEY_ID     = credentials('jenkins-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
            }        
        steps {

                echo "Login on ECR"
                sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 966508915346.dkr.ecr.us-east-1.amazonaws.com"       		
	            echo 'Login Completed' 
                echo "Pushing docker image to ECR with current build tag"
                sh " docker tag ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ${DOCKER_REGISTRY_NAME}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                sh " docker push ${DOCKER_REGISTRY_NAME}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                echo 'Pushing docker image with tag latest'
                sh "docker tag ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ${DOCKER_REGISTRY_NAME}/${DOCKER_IMAGE_NAME}:latest"
                sh "docker push ${DOCKER_REGISTRY_NAME}/${DOCKER_IMAGE_NAME}:latest"
            }
        }
        
        
       stage("Deploy to Production"){
        when {
                branch 'main'
            }
             steps {              

              sh ("""
	          kubectl delete -f account-portal.yaml                 
                  kubectl apply -f account-portal.yaml
                """)
                
             }
         }
    }
}
