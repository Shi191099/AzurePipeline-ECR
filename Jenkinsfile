pipeline {
agent any
environment {
    AWS_CREDENTIALS= credentials('AWS')
    AWS_ACCOUNT_ID="805392809179"             
    AWS_DEFAULT_REGION="ap-south-1" 
    IMAGE_REPO_NAME="oz-ecr"
    REPOSITORY_URI= "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"

}   
    stages {
        stage('Code checkout') {
            steps {
                script {
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/demo-organization-01/ozontel.git']]])
                }
            }
        } 
        stage('Login to AWS ECR') {
            steps {
                script {
                 sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 805392809179.dkr.ecr.ap-south-1.amazonaws.com'
                 //sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                 // sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 440883647063.dkr.ecr.us-east-1.amazonaws.com"
                }
            }
        }
        stage('Building Docker Image') {
            steps {
                script {
                  sh "docker build -t ${REPOSITORY_URI}:latest ."
               }
            }
       }
        stage('Pushing Docker image into ECR') {
            steps {
               script {
                sh "docker push ${REPOSITORY_URI}:latest"
               }
            }
        }
     // Stopping Docker containers for cleaner Docker run
        stage('stop previous containers') {
            steps {
               sh 'docker ps -f name=oz-app -q | xargs --no-run-if-empty docker container stop'
               sh 'docker container ls -a -fname=oz-app -q | xargs -r docker container rm'
         }
       }

        stage('Deploy') {
            steps{
                script {
                  sh "docker run -d -p 8082:8080 --rm --name oz-app ${REPOSITORY_URI}:latest"
                }
            }
        } 
    
    }
} 
