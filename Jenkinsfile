pipeline {
    agent any
    environment {
       AWS_ACCOUNT_ID="867957089729"
       AWS_DEFAULT_REGION="us-east-1"
       IMAGE_REPO_NAME="jenkins-pipeline-build-demo"
       IMAGE_TAG="latest"
       REPOSITORY_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
     }

     stages {

         stage( ' Logging into AWS ECR') {
           steps {
               script {
               sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"

                }

            }
        }

        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/m-k-thakur/deploy_nodeJs.git']]])
            }
         }

     //Building docker images
      stage('Building image') {
        steps{
          script {
            dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
            }
          }
        }
    //Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
      steps{
         script {
                sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
         }
        }
       }
	stage('Apply Kubernetes Files') {
      steps {
          withKubeConfig([credentialsId: 'cred', serverUrl: 'https://B1639BD95D7515C624C499A989414455.gr7.us-east-1.eks.amazonaws.com']) {
 //         sh 'cat deployment.yaml | sed "s/{{BUILD_NUMBER}}/$BUILD_NUMBER/g" | kubectl apply -f -'
            sh '/home/ec2-user/kubectl delete --all pods'
            sh '/home/ec2-user/kubectl apply -f eks_cicd/deployment.yaml'
            sh '/home/ec2-user/kubectl apply -f eks_cicd/service.yaml'

        }
      }
     }
    }
}