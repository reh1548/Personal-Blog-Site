pipeline {
  agent any

  environment {
    DOCKER_IMAGE_NAME = 'my-flask-app'
    DOCKER_CONTAINER_NAME = 'my-flask-container'
    CONTAINER_PORT = 5000
    HOST_PORT = 80  // Change to desired host port if not using random
  }

  stages {
    stage('Clone Repository') {
      steps {
        git 'https://github.com/reh1548/Personal-Blog-Site.git'
      }
    }
    stage('Copy .env') {
      steps {
        sh 'cp /home/rahmed/Documents/.env ${WORKSPACE}/.env'
      }
    }
    stage('Build') {
      steps {
        sh 'docker build -t $DOCKER_IMAGE_NAME .'
      }
    }
    stage('Test') {
      steps {
        sh 'docker run $DOCKER_IMAGE_NAME python -m pytest app/tests/'
      }
    }
    stage('Deploy') {
      steps {
        sh 'docker stop $DOCKER_CONTAINER_NAME || true'
        sh 'docker rm -f $DOCKER_CONTAINER_NAME || true'
        sh 'docker run -d -p $HOST_PORT:$CONTAINER_PORT --env-file .env --name $DOCKER_CONTAINER_NAME $DOCKER_IMAGE_NAME'
      }
    }
  }

  post {
    always {
      sh 'docker logout'
    }
  }
}
