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
              https://api.github.com/repos/reh1548/Personal-Blog-Site/actions/secrets | \
              jq -r '.secrets[] | "\\(.name)=\\(.value)"' > .env
            ''', returnStdout: true).trim()
            echo response
          }
        }
      }
    }
    stage('Build') {
      steps {
        sh 'docker build -t ${DOCKER_IMAGE_NAME} .'
      }
    }
    stage('Test') {
      steps {
        sh 'docker run --rm --env-file .env ${DOCKER_IMAGE_NAME} python -m pytest app/tests/'
      }
    }
    stage('Deploy') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'DOCKER_REGISTRY_CREDS', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
          script {
            sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
            sh 'docker push ${DOCKER_IMAGE_NAME}'
            sh 'docker stop ${DOCKER_CONTAINER_NAME} || true'
            sh 'docker rm -f ${DOCKER_CONTAINER_NAME} || true'
            sh 'docker run -d -p ${HOST_PORT}:${CONTAINER_PORT} --env-file .env --name ${DOCKER_CONTAINER_NAME} ${DOCKER_IMAGE_NAME}'
          }
        }
      }
    }
  }

  post {
    always {
      sh 'docker logout'
    }
  }
}
