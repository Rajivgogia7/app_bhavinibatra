pipeline {
   
  agent any
  
   environment{
       username = 'bhavinibatra'
       registry = 'bhavinibatra/develop'
    }

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM', branches: [
          [name: '*/develop']
        ], userRemoteConfigs: [
          [credentialsId: 'GitCreds', url: 'https://github.com/BhaviniB/app_bhavinibatra']
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
             
            bat "docker build -t i-${username}-develop --no-cache ."
        
      }
    }
    stage('Containers'){
      parallel{
        stage('Pre-container check'){
          steps{
            bat 'docker rm -f c-bhavinibatra-develop'
            
          }
        }
      stage('Move image to DockerHub') {
      steps {
             bat "docker tag i-${username}-develop ${registry}:${BUILD_NUMBER}"
              bat "docker tag i-${username}-develop ${registry}:latest"
             withDockerRegistry([credentialsId: 'DockerHub', url:""]){
             bat "docker push ${registry}:${BUILD_NUMBER}"
             bat "docker push ${registry}:latest"
             }
        
      }
    }
      }
    }

     stage('Docker Deployment') {
      steps {
             bat "docker run --name c-${username}-develop -d -p 7300:80 ${registry}:${BUILD_NUMBER}"
            
      }
    }
    stage('Kubernetes Deployment'){
      steps{
        bat "kubectl apply -f deployment.yaml"
      }
    }


  }

}