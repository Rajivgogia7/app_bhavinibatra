pipeline {
   
  agent any
  
   environment{
       scannerHome = tool 'sonar_scanner_dotnet'
       username = 'bhavinibatra'
       registry = 'bhavinibatra/test-master'
       project_id = 'test-project-321516'
       cluster_name = 'test-cluster'
       location = 'us-central1-c'
       credentials_id = 'test-project'
    }

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM', branches: [
          [name: '*/master']
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
                stage('Start Sonar Analysis') {
      steps {
        withSonarQubeEnv('Test_Sonar'){
            bat "${scannerHome}\\SonarScanner.MSBuild.exe begin /k:WebApplication4 /n:WebApplication4 /v:1.0"
        }
        
      }
    }

    stage('Build') {
      steps {
        bat 'dotnet clean WebApplication4\\WebApplication4.csproj'

        bat 'dotnet build WebApplication4\\WebApplication4.csproj -c Release -o "WebApplication4/app/build"'

      }
    }
    stage('Automated Unit Testing') {
      steps {
        bat 'dotnet test .'

      }
    }
        
         stage('End Sonar Analysis') {
      steps {
        withSonarQubeEnv('Test_Sonar'){
            bat "${scannerHome}\\SonarScanner.MSBuild.exe end"
        }
        
      }
    }
    
      stage('Build Docker Image') {
      steps {
             bat "dotnet publish WebApplication4\\WebApplication4.csproj"
             
            bat "docker build -t i-${username}-mastertest --no-cache ."
        
      }
    }
     
      stage('Move image to DockerHub') {
      steps {
             bat "docker tag i-${username}-mastertest ${registry}:${BUILD_NUMBER}"
              bat "docker tag i-${username}-master ${registry}:latest"
             withDockerRegistry([credentialsId: 'DockerHub', url:""]){
             bat "docker push ${registry}:${BUILD_NUMBER}"
              bat "docker push ${registry}:latest"
             }
        
      }
    }
     stage('Docker Deployment') {
      steps {
             bat "docker run --name c-${username}-mastertest -d -p 7800:80 ${registry}:${BUILD_NUMBER}"
            
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