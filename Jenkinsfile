pipeline {
  agent any

  environment {
    DOCKER_IMAGE_NAME = 'my-flask-app'
    DOCKER_CONTAINER_NAME = 'my-flask-container'
    CONTAINER_PORT = 5000
    HOST_PORT = 80
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Fetch GitHub Secrets') {
      steps {
        withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')]) {
          script {
            def response = sh(script: '''
              curl -H "Authorization: token ${GITHUB_TOKEN}" \
              https://api.github.com/repos/reh1548/Personal-Blog-Site/actions/secrets \
              | jq -r '.secrets[] | "\(.name)=\(.value)"' > .env
            ''', returnStdout: true).trim()
            echo response
          }
        }
      }
    }
    stage('Build') {
      steps {
        sh 'docker build -
