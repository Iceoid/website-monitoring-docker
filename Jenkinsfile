pipeline {
  environment {
    userName = "hexlo"
    imageName = "website-monitoring-docker"
    tag = ":latest"
    gitRepo = "https://github.com/${userName}/${imageName}.git"
    dockerhubRegistry = "${userName}/${imageName}"
    githubRegistry = "ghcr.io/${userName}/${imageName}"
    
    dockerhubCredentials = 'DOCKERHUB_TOKEN'
    githubCredentials = 'GITHUB_TOKEN'
    
    dockerhubImage = ''
    dockerhubImageLatest = ''
    githubImage = ''
  }
  agent any
  stages {
    stage('Cloning Git') {
      steps {
        git branch: 'main', credentialsId: "${githubCredentials}", url: "${gitRepo}"
      }
    }
    stage('Building image') {
      steps{
        script {
          dockerhubImageLatest = docker.build( "${dockerhubRegistry}${tag}" ) 
          
          githubImage = docker.build( "${githubRegistry}${tag}" )
        }
      }
    }
    stage('Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', "${dockerhubCredentials}" ) {
            dockerhubImageLatest.push()
          }
          docker.withRegistry("https://${githubRegistry}", "${githubCredentials}" ) {
            githubImage.push()
          }
        }
      }
    }
    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi ${dockerhubRegistry}${tag}"
        sh "docker rmi ${githubRegistry}${tag}"
      }
    }
  }
  post {
    failure {
        mail bcc: '', body: "<b>Jenkins Build Report</b><br><br> Project: ${env.JOB_NAME} <br> \
        Build Number: ${env.BUILD_NUMBER} <br> \
        Status: <b>Failed</b> <br> \
        Build URL: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', \
        subject: "Jenkins Build Failed: ${env.JOB_NAME}", to: "jenkins@mindlab.dev";  

    }
  }
}
