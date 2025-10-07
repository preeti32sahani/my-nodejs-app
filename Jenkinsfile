pipeline {
  agent any

  environment {
    DOCKER_REGISTRY = 'preetisahani123'
    IMAGE_NAME = 'my-nodejs-app'
  }

  stages {

    stage('Checkout Source') {
      steps {
        git url: 'https://github.com/preeti32sahani/my-nodejs-app.git', branch: 'main'
      }
    }

    stage('Build & Push Docker Images') {
      steps {
        script {
          withCredentials([usernamePassword(
            credentialsId: 'docker-hub-creds',
            usernameVariable: 'DOCKER_USERNAME',
            passwordVariable: 'DOCKER_PASSWORD')]) {

            // Use a cmd-friendly login form on Windows agents
            bat "docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%"
          }

          // Build explicitly so we can tag correctly
          bat "docker build -t ${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER} ."

          // Push the built image
          bat "docker push ${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER}"
        }
      }
    }

    stage('Deploy Multi-Container App') {
      steps {
        script {
          bat """
            docker stop node-web node-mongo-db || exit 0
            docker rm node-web node-mongo-db || exit 0
            set BUILD_NUMBER=${env.BUILD_NUMBER} && docker-compose up -d --force-recreate
          """
        }
      }
    }
  }

  post {
    always {
      cleanWs()
    }
  }
}
