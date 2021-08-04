pipeline {
   
  agent any
  
   environment{
       username = 'bhavinibatra'
       registry = 'bhavinibatra/test-develop'
       project_id = 'test-project-321516'
       cluster_name = 'test-cluster'
       location = 'us-central1-c'
       credentials_id = 'test-project'
    }

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM', branches: [
          [name: '*/develop']
        ], userRemoteConfigs: [
          [credentialsId: 'GitCreds', url: 'https://github.com/BhaviniB/test']
        ]])
      }
    }
    stage('Restore packages') {
      steps {
        bat "dotnet restore WebApplication4\\WebApplication4.csproj"
      }
    }

    stage('Build') {
      steps {
        bat 'dotnet clean WebApplication4\\WebApplication4.csproj'

        bat 'dotnet build WebApplication4\\WebApplication4.csproj -c Release -o "WebApplication4/app/build"'

      }
    }

     stage('Release artifact') {
      steps {
             bat "dotnet publish WebApplication4\\WebApplication4.csproj"

      }
    }
       
      stage('Build Docker Image') {
      steps {
             
            bat "docker build -t i-${username}-developtest --no-cache ."
        
      }
    }

    
     
      stage('Move image to DockerHub') {
      steps {
             bat "docker tag i-${username}-developtest ${registry}:${BUILD_NUMBER}"
              bat "docker tag i-${username}-developtest ${registry}:latest"
             withDockerRegistry([credentialsId: 'DockerHub', url:""]){
             bat "docker push ${registry}:${BUILD_NUMBER}"
             bat "docker push ${registry}:latest"
             }
        
      }
    }
     stage('Docker Deployment') {
      steps {
             bat "docker run --name c-${username}-developtest -d -p 7700:80 ${registry}:${BUILD_NUMBER}"
            
      }
    }
      stage('Docker to GKE') {
         steps{
             step([$class: 'KubernetesEngineBuilder',
             projectId: env.project_id, 
             clusterName: env.cluster_name,
             location: env.location, 
             manifestPattern: 'Deployment.yaml', 
             credentialsId: env.credentials_id, 
             verifyDeployments: true])
         }
      
    }


  }

}