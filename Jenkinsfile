pipeline {
   
  agent any
  
   environment{
       scannerHome = tool 'sonar_scanner_dotnet'
       username = 'bhavinibatra'
       registry = 'bhavinibatra/nagp-jenkins-1'
    }

  stages {
    stage('Checkout') {
      steps {
        checkout([$class: 'GitSCM', branches: [
          [name: '*/master']
        ], userRemoteConfigs: [
          [credentialsId: 'GitCreds', url: 'https://github.com/BhaviniB/nagp-jenkins-1']
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
             
            bat "docker build -t i-${username}-master --no-cache ."
        
      }
    }
     
      stage('Move image to DockerHub') {
      steps {
             bat "docker tag i-${username}-master ${registry}:${BUILD_NUMBER}"
             withDockerRegistry([credentialsId: 'DockerHub', url:""]){
             bat "docker push ${registry}:${BUILD_NUMBER}"
             }
        
      }
    }
     stage('Docker Deployment') {
      steps {
             bat "docker run --name c-${username}-master -d -p 7700:80 ${registry}:${BUILD_NUMBER}"
            
      }
    }


  }

}