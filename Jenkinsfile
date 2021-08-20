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
          [credentialsId: 'GitCreds', url: 'https://github.com/Rajivgogia7/app_bhavinibatra']
        ]])
      }
    }
    stage('Nuget Restore') {
      steps {
        bat "dotnet restore WebApplication4\\WebApplication4.csproj"
      }
    }

    stage('Code Build') {
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
       
      stage('Docker Image') {
      steps {
             
            bat "docker build -t i-${username}-develop:${BUILD_NUMBER} --no-cache ."
        
      }
    }
    stage('Containers'){
      parallel{
  stage('PreContainerCheck') {
                    environment {
                        CONTAINER_ID = "${bat(script:'docker ps -aqf name="^c-bhavinibatra-develop$"', returnStdout: true).trim().readLines().drop(1).join("")}"
                    }
                    steps {
                        echo "Running pre container check"
                        script {
                        if(env.CONTAINER_ID != null) {
                            echo "Removing container: ${env.CONTAINER_ID}"
                            bat "docker rm -f ${env.CONTAINER_ID}"       
                        }
                   }
                                  
                    }
                }  
      stage('Push to DockerHub') {
      steps {
             bat "docker tag i-${username}-develop:${BUILD_NUMBER} ${registry}:${BUILD_NUMBER}"
              bat "docker tag i-${username}-develop:latest ${registry}:latest"
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