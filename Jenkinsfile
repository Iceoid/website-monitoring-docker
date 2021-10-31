pipeline {
  environment {
    gitRepo = "https://github.com/Iceoid/website-monitoring-docker.git"
    imageName = "website-monitoring-docker"
    dockerhubRegistry = "iceoid/website-monitoring"
    githubRegistry = "ghcr.io/iceoid/$imageName"
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
        git branch: 'main', credentialsId: 'GITHUB_TOKEN', url: gitRepo
      }
    }
    stage('Building image') {
      steps{
        script {
          dockerhubImageLatest = docker.build dockerhubRegistry + ":latest"
          
          githubImage = docker.build githubRegistry + ":latest"
        }
      }
    }
    stage('Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', dockerhubCredentials ) {
            dockerhubImageLatest.push()
          }
          docker.withRegistry('https://' + githubRegistry, githubCredentials) {
            githubImage.push()
          }
        }
      }
    }
    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $dockerhubRegistry:latest"
        sh "docker rmi $githubRegistry:latest"
      }
    }
  }
  post {
    failure {
        mail bcc: '', body: "<b>Jenkins Build Report</b><br><br> Project: ${env.JOB_NAME} <br> \
        Build Number: ${env.BUILD_NUMBER} <br> \
        Status: <b>Failed</b> <br> \
        Build URL: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', \
        subject: "Jenkins Build Failed: ${env.JOB_NAME}", to: "alerts@mindlab.dev";  

    }
  }
}
