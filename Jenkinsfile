pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="tbcommerce"
        AWS_DEFAULT_REGION="us-east-1" 
	CLUSTER_NAME="Nodejs"
	SERVICE_NAME="nodejsapp-service"
	TASK_DEFINITION_NAME="nodejs-app"
	DESIRED_COUNT="1"
        IMAGE_REPO_NAME="nodesample"
        IMAGE_TAG="${env.BUILD_ID}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
	registryCredential = "176295807911"
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
        
    // Build the Docker image from the Dockerfile in "WebUi" directory
      stage('Docker build') {
     // dir('WebUI') {
        sh "docker build --no-cache -t ${env.IMAGE_REPO_NAME}:${IMAGE_TAG} ."
      //}
      }
   
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
			docker.withRegistry("https://" + REPOSITORY_URI, "ecr:${AWS_DEFAULT_REGION}:" + registryCredential) {
                    	dockerImage.push()
                	}
         }
        }
      }
      
    stage('Deploy') {
     steps{
            withAWS(credentials: registryCredential, region: "${AWS_DEFAULT_REGION}") {
                script {
			sh './script.sh'
                }
            } 
        }
      }      
      
    }
}

