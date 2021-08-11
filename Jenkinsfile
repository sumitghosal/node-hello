pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="772662843713"
        AWS_DEFAULT_REGION="us-east-1" 
        IMAGE_REPO_NAME="jenkinstest"
        IMAGE_TAG="Latest"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
   
    stages {
        
         stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
                 
            }
        }
        
        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '13567', url: 'https://github.com/sumitghosal/sgldevops.git']]]) 
            }
        }
  
  
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
    }
   
    
    stage('Pushing to ECR') {
     steps{  
         script {
                sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
         }
        }
      } 
        
      stage('kubernet cluster creation') {  
         steps {
           script {
                   sh "/usr/local/bin/eksctl create cluster --name myappcluster --region us-east-1 --zones us-east-1a,us-east-1c --nodegroup-name mynodes --node-type t3.small --managed"
                   sh "/usr/local/bin/kubectl create clusterrolebinding cluster-system-anonymous --clusterrole=cluster-admin --user=system:anonymous"
    }        
}     

        }
         stage ('K8S Deploy') {
          steps {
             script {
                 sh "/usr/local/bin/kubectl get nodes"
                 sh "/usr/local/bin/kubectl apply -f ${WORKSPACE}/kubernatesdeploy.yaml"
        }
       }
     }
    }
} 

