pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="018416181156"
        AWS_DEFAULT_REGION="ap-south-1" 
	CLUSTER_NAME="default"
	SERVICE_NAME="nodejs-container-service"
	TASK_DEFINITION_NAME="first-run-task-definition"
	DESIRED_COUNT="1"
        IMAGE_REPO_NAME="syed-ecr"
        IMAGE_TAG="${env.BUILD_ID}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
        registryCredential = "demo-admin-user"
    }
   
    stages {

    // Tests
    stage('Unit Tests') {
        steps{
            script {
                sh 'npm install'
                sh 'npm test -- --watchAll=false'
            }
        }
    }
        
    // Building Docker images
    stage('Building image') {
        steps{
            script {
                dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
            }
        }
    }
   
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
        steps {  
            script {
                withCredentials([
                    [
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: registryCredential,
                        accessKeyVariable: 'AKIAQISNQT6SOS63VTVT',
                        secretKeyVariable: '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',
                    ]
                ]) {
                    docker.withRegistry("https://${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com", 'ecr') {
                        dockerImage.push()
                    }
                }
            }
        }
    }
      
    stage('Deploy') {
        steps{
            withAWS(credentials: registryCredential, region: "${AWS_DEFAULT_REGION}") {
                script {
                    sh 'chmod +x script.sh'
                }
            } 
        }
    }      
      
  }
}

