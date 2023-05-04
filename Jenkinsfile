pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="176295807911"
        AWS_DEFAULT_REGION="us-east-1" 
	CLUSTER_NAME="Nodejs"
	SERVICE_NAME="nodejsapp-service"
	TASK_DEFINITION_NAME="nodejs-app"
	DESIRED_COUNT="1"
        IMAGE_REPO_NAME="nodesample"
        IMAGE_TAG="${env.BUILD_ID}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
// 	registryCredential = "tbcommerce"
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
        
    stage("Build Docker image") {
        steps {
            sh "docker build -t ${env.IMAGE_REPO_NAME}:${env.IMAGE_TAG} ."
        }
    }
   
    // Uploading Docker images into AWS ECR
    stage("Push Docker image to ECR") {
        steps {
            withCredentials([
                [
                    $class: 'AmazonWebServicesCredentialsBinding',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
                    credentialsId: '176295807911'
                ]
            ]) {
                sh "aws ecr get-login-password --region ${env.AWS_REGION} | docker login --username AWS --password-stdin ${env.DOCKER_IMAGE}"
                sh "docker push ${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}"
            }
        }
    }

    stage("Update ECS service") {
        steps {
            withCredentials([
                [
                    $class: 'AmazonWebServicesCredentialsBinding',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
                    credentialsId: '176295807911'
                ]
            ]) {
                sh "aws ecs update-service --cluster ${env.ECS_CLUSTER} --service ${env.ECS_SERVICE} --force-new-deployment --image ${env.DOCKER_IMAGE}:${env.BUILD_NUMBER}"
            }
        }
    }
}
}

