/*
pipeline {

    environment {
    dockerimagename = "jemimaht/nodeapp"
    dockerImage = ""
   }

  agent any
 

  stages {

    stage('Checkout Source') {
      steps {
      git branch: 'main', url: 'https://github.com/JimmyT96/nodeapp_test.git'
      }
    }

    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }

    stage('Pushing Image') {
      environment {
               registryCredential = 'dockerhublogin'
           }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("latest")
          }
        }
      }
    }

    stage("kubernetes deployment"){
      steps {
        script {
        sh 'kubectl apply -f deploymentservice.yml'
         
        }
      }
}
}

  }
*/

pipeline {
    environment {
        // Use the build number for unique tagging (2026 Best Practice)
        DOCKER_REGISTRY = "docker.io"
        DOCKER_REPO = "jemimaht/nodeapp"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        FULL_IMAGE_NAME = "${DOCKER_REPO}:${IMAGE_TAG}"
        
        // AWS/EKS Credentials IDs from your Jenkins store
        DOCKER_HUB_CREDS = 'dockerhublogin'
        AWS_CREDS_ID = 'aws-credentials-id' 
        EKS_CLUSTER_NAME = 'your-eks-cluster-name'
        AWS_REGION = 'us-east-1'
    }

    agent any

    stages {
        stage('Checkout Source') {
            steps {
                git branch: 'main', url: 'https://github.com/JimmyT96/nodeapp_test.git'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    // Build the image with the specific build number tag
                    dockerImage = docker.build("${FULL_IMAGE_NAME}")
                }
            }
        }

        stage('Pushing Image') {
            steps {
                script {
                    docker.withRegistry("https://index.docker.io/v1/", DOCKER_HUB_CREDS) {
                        // Push both the specific build tag and 'latest' for convenience
                        dockerImage.push("${IMAGE_TAG}")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                // withAWS or withCredentials ensures secure access to your EKS cluster
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDS_ID]]) {
                    script {
                        // 1. Update local kubeconfig to point to EKS
                        sh "aws eks update-kubeconfig --name ${EKS_CLUSTER_NAME} --region ${AWS_REGION}"
                        
                        // 2. Apply the manifest (Service and Deployment structure)
                        sh "kubectl apply -f deploymentservice.yml"
                        sh "kubectl apply -f deployment.yml"
                        
                        // 3. Force update the image in the deployment to trigger a rolling restart
                        // Replace 'springboot' with the container name defined in your deployment.yml
                        sh "kubectl set image deployment/springboot springboot=${FULL_IMAGE_NAME} --record"
                        
                        // 4. Verify the rollout status
                        sh "kubectl rollout status deployment/springboot"
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Cleanup: remove the local image to save disk space on the Jenkins agent
            sh "docker rmi ${FULL_IMAGE_NAME} || true"
        }
    }
}
