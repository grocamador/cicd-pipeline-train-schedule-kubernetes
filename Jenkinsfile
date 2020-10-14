pipeline {
    agent any
    
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "grocamador/train-schedule"
        CHKP_CLOUDGUARD_ID = credentials("ivan-user")
        CHKP_CLOUDGUARD_SECRET = credentials("ivan-pass")
        SG_CLIENT_ID = credentials("sg-client")
        SG_SECRET_KEY = credentials("sg-secret")

        }
    
    stages {
        
       stage('ShiftLeft Code Scan') {   
            steps {   
                   script {      
                        try {
                            sh 'chmod +x shiftleft' 
                            sh './shiftleft code-scan -s .'
                        } catch (Exception e) {
                            echo "Request for Approval"  
                         }
                   }
            }
         
         }
        
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        
       stage("Deploy to Production"){
            when {
                branch 'master'
            }
            steps { 
                kubernetesDeploy kubeconfigId: 'kubeconfig', configs: 'deploy.yml', enableConfigSubstitution: true  // REPLACE kubeconfigId
 
             }
            post{
                success{
                    echo "Successfully deployed to Production"
                }
                failure{
                    echo "Failed deploying to Production"
                }
            }
        }
  
    }
}
