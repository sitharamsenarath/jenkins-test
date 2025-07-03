pipeline {
  agent any
  environment {
    IMAGE = "ghcr.io/sitharamsenarath/jenkins-test"
    GHCR_PAT = credentials('DOCKER_IMAGE')  // ID of the token we’ll add
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $IMAGE .'
      }
    }
    stage('Push to GHCR') {
      steps {
        sh 'echo $GHCR_PAT | docker login ghcr.io -u sitharamsenarath --password-stdin'
        sh 'docker push $IMAGE'
      }
    }
  }
}
