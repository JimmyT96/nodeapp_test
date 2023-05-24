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
