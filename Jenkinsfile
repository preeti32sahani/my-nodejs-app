pipeline {
  agent any
  environment {
    DOCKER_REGISTRY = 'bunny011' // <-- Replace this
  }
  stages {
    stage('Checkout Source') {
      steps {
        git url: 'https://github.com/loyalbunny/my-nodejs-app.git', branch: 'main'
      }
    }

    stage('Build & Push Docker Images') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'docker-hub-creds',
                                             usernameVariable: 'DOCKER_USERNAME',
                                             passwordVariable: 'DOCKER_PASSWORD')]) {
            // Use bat instead of sh on Windows
            bat "docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%"
            bat "docker-compose build web"
            def imageTag = "${DOCKER_REGISTRY}/my-nodejs-app:${env.BUILD_NUMBER}"
            bat "docker tag my-nodejs-app:latest ${imageTag}"
            bat "docker push ${imageTag}"
          }
        }
      }
    }

    stage('Deploy Multi-Container App') {
      steps {
        script {
          bat "docker stop node-web node-mongo-db || exit 0"
          bat "docker rm node-web node-mongo-db || exit 0"
          bat "set BUILD_NUMBER=${env.BUILD_NUMBER} && docker-compose up -d --force-recreate"
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
