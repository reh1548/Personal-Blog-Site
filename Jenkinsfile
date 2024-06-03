pipeline {
  agent any

  environment {
    DOCKER_IMAGE_NAME = 'my-flask-app'
    DOCKER_CONTAINER_NAME = 'my-flask-container'
    CONTAINER_PORT = 5000
    HOST_PORT = 80  // Change to desired host port if not using random
    APP_API_KEY = "${env.APP_API_KEY}"
    MY_MAIL = "${env.MY_MAIL}"
    MY_MAIL_PASSWORD = "${env.MY_MAIL_PASSWORD}"
    RECIPIENT_MAIL = "${env.RECIPIENT_MAIL}"
    DATABASE_URL = "${env.DATABASE_URL}"
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
        sh 'docker run my-flask-app python -m pytest app/tests/'
      }
    }
    stage('Deploy') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
          sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
          sh 'docker push $DOCKER_BFLASK_IMAGE'
          sh 'docker stop $DOCKER_CONTAINER_NAME || true'
          sh 'docker rm -f $DOCKER_CONTAINER_NAME || true'
          sh 'docker run -d -p ${HOST_PORT}:${CONTAINER_PORT} --name $DOCKER_CONTAINER_NAME $DOCKER_IMAGE_NAME'
        }
    }
  }

  post {
    always {
      sh 'docker logout'
    }
  }
}
