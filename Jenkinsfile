pipeline {
  agent any

  environment {
    DOCKER_IMAGE_NAME = 'my-flask-app'
    DOCKER_CONTAINER_NAME = 'my-flask-container'
    CONTAINER_PORT = 5000
    HOST_PORT = 80  // Change to desired host port if not using random
  }

  stages {
    stage('Build') {
      steps {
        sh 'docker build -t my-flask-app .'
        sh 'docker tag my-flask-app $DOCKER_IMAGE_NAME'
      }
    }
    stage('Test') {
      steps {
        withCredentials([string(credentialsId: 'DATABASE_URL', variable: 'DATABASE_URL'),
                         string(credentialsId: 'APP_API_KEY', variable: 'APP_API_KEY'),
                         string(credentialsId: 'MY_MAIL', variable: 'MY_MAIL'),
                         string(credentialsId: 'MY_MAIL_PASSWORD', variable: 'MY_MAIL_PASSWORD'),
                         string(credentialsId: 'RECIPIENT_MAIL', variable: 'RECIPIENT_MAIL')]) {
          sh 'docker run -e DATABASE_URL=$DATABASE_URL -e APP_API_KEY=$APP_API_KEY -e MY_MAIL=$MY_MAIL -e MY_MAIL_PASSWORD=$MY_MAIL_PASSWORD -e RECIPIENT_MAIL=$RECIPIENT_MAIL my-flask-app python -m pytest app/tests/'
        }
      }
    }
    stage('Deploy') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
          sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
          sh 'docker push $DOCKER_BFLASK_IMAGE'
          sh 'docker stop $DOCKER_CONTAINER_NAME || true'
          sh 'docker rm -f $DOCKER_CONTAINER_NAME || true'
          withCredentials([string(credentialsId: 'DATABASE_URL', variable: 'DATABASE_URL'),
                           string(credentialsId: 'APP_API_KEY', variable: 'APP_API_KEY'),
                           string(credentialsId: 'MY_MAIL', variable: 'MY_MAIL'),
                           string(credentialsId: 'MY_MAIL_PASSWORD', variable: 'MY_MAIL_PASSWORD'),
                           string(credentialsId: 'RECIPIENT_MAIL', variable: 'RECIPIENT_MAIL')]) {
            sh 'docker run -d -p ${HOST_PORT}:${CONTAINER_PORT} --name $DOCKER_CONTAINER_NAME -e DATABASE_URL=$DATABASE_URL -e APP_API_KEY=$APP_API_KEY -e MY_MAIL=$MY_MAIL -e MY_MAIL_PASSWORD=$MY_MAIL_PASSWORD -e RECIPIENT_MAIL=$RECIPIENT_MAIL $DOCKER_IMAGE_NAME'
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
