pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = 'my-flask-app'
        DOCKER_CONTAINER_NAME = 'my-flask-container'
        CONTAINER_PORT = 5000
        HOST_PORT = 80  // Change to desired host port if not using random
    }

    stages {
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
                sh 'docker-compose build'
            }
        }
        stage('Test') {
            steps {
                withCredentials([
                    file(credentialsId: 'ENV_FILE', variable: 'ENV_FILE')
                ]) {
                    sh 'docker-compose run --env-file .env flask-app python -m pytest app/tests/'
                }
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'DOCKER_REGISTRY_CREDS', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')
                ]) {
                    sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
                    sh 'docker-compose push'
                    sh 'docker-compose down'
                    sh '''
                    docker-compose up -d --env-file .env
                    '''
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
