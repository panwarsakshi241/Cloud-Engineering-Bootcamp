pipeline {
  agent any 
  environment{
      registry = "256707027315.dkr.ecr.eu-north-1.amazonaws.com/flask-hello-world"
      }
  stages {
    stage('checkout') {
      steps {
        checkout scmGit(branches: [[name:'*/main']], extensions: [], userRemoteConfigs:[[credentialsId:'github-cred',
        url: 'https://github.com/panwarsakshi241/Cloud-Engineering-Bootcamp.git']])
      }
    }
    // Building Docker images
    stage('Building image') {
      steps {
        script { dockerImage = docker.build registry }
      }
    }
    // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
      steps {
        script {
          sh 'aws ecr get-login-password --region eu-north-1 | docker login --username  AWS --password-stdin 256707027315.dkr.ecr.eu-north-1.amazonaws.com' 
          sh 'docker push 256707027315.dkr.ecr.eu-north-1.amazonaws.com/flask-hello-world:latest'
        }
      }
    }
    stage('Docker Run') {
      steps {
        script {
          sh '''
            docker rm -f myflask-container || true
            docker run -d -p 8096:5000 --rm --name myflask-container 256707027315.dkr.ecr.eu-north-1.amazonaws.com/flask-hello-world:latest
        '''
        }
      }
    }
  }
}