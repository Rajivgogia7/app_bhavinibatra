pipeline {
   
  agent any
  
   environment{
       scannerHome = tool 'sonar_scanner_dotnet'
       username = 'bhavinibatra'
       registry = 'bhavinibatra/master'
    }

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM', branches: [
          [name: '*/master']
        ], userRemoteConfigs: [
          [credentialsId: 'GitCreds', url: 'https://github.com/BhaviniB/app_bhavinibatra']
        ]])
      }
    }
    stage('Nuget Restore') {
      steps {
        bat "dotnet restore WebApplication4\\WebApplication4.csproj"
      }
    }
      stage('Start Sonarqube Analysis') {
      steps {
        withSonarQubeEnv('Test_Sonar'){
            bat "${scannerHome}\\SonarScanner.MSBuild.exe begin /k:sonar-bhavinibatra /n:sonar-bhavinibatra /v:1.0"
        }
        
      }
    }

    stage('Code Build') {
      steps {
        bat 'dotnet clean WebApplication4\\WebApplication4.csproj'

        bat 'dotnet build WebApplication4\\WebApplication4.csproj -c Release -o "WebApplication4/app/build"'

      }
    }
        
         stage('Stop Sonarqube Analysis') {
      steps {
        withSonarQubeEnv('Test_Sonar'){
            bat "${scannerHome}\\SonarScanner.MSBuild.exe end"
        }
        
      }
    }
    
      stage('Docker Image') {
      steps {
             bat "dotnet publish WebApplication4\\WebApplication4.csproj"
             
            bat "docker build -t i-${username}-master:${BUILD_NUMBER} --no-cache ."
        
      }
    }
     
    stage('Containers'){
      parallel{
        stage('Pre-container check'){

          environment{
          containerId = ${bat(script:'docker ps -aqf "name=^c-bhavinibatra-master$"', returnStdout:true).trim().readLines()}
          }
          steps{
            script {
                    if (containerId != null) {
            bat 'docker rm -f c-bhavinibatra-master && echo "container c-bhavinibatra-master removed" || echo "container c-bhavinibatra-master does not exist"'
                    } else {
                        echo 'No container removal needed.'
                    }
                }
          }
        }
      stage('Push to DockerHub') {
      steps {
             bat "docker tag i-${username}-master:${BUILD_NUMBER} ${registry}:${BUILD_NUMBER}"
              bat "docker tag i-${username}-master:latest ${registry}:latest"
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
             bat "docker run --name c-${username}-master -d -p 7200:80 ${registry}:${BUILD_NUMBER}"
            
      }
    }
    stage('Kubernetes Deployment'){
      steps{
        bat "kubectl apply -f deployment.yaml"
      }
    }

  }

}