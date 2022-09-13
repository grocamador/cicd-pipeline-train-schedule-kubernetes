pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "grocamador/cicd-demo"
        }
    
stages {

/*
        stage('Test') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }

*/

    stage('Build Docker Image') {
            when {
                branch 'master'
            }
        steps {
                echo 'Building docker image'
                sh "docker build -t ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ."
            }
        }


    stage('Push Docker Image') {
        when {
            branch 'master'
        }
        steps {

                echo 'Pushing docker image with current build tag'
                sh " docker push ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                echo 'Pushing docker image with current latest'
                sh "docker push ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
            }
        }
            


    stage('Deploy to stage') {
        when {
            branch 'master'
        }
            steps {
            sh ("""     
                  kubectl delete -f train-schedule-kube-stage.yml
                  kubectl apply -f train-schedule-kube-stage.yml
                """)
 
            }
        }
        
       stage("Deploy to Production"){
        when {
                branch 'master'
            }
             steps {              
                input 'Deploy to Production?'
                milestone(1)   
              sh ("""                
                  echo \$KUBECONFIG
                  kubectl delete -f train-schedule-kube.yml
                  kubectl apply -f train-schedule-kube.yml
                """)
                
             }
         }
    }
}
