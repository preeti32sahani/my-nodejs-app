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
            def imageTag = "${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
            bat "docker tag ${IMAGE_NAME}:${env.BUILD_NUMBER} ${imageTag}"
            bat "docker push ${imageTag}"
          }
        }
      }
    }

   stage('Deploy Multi-Container App') {
            steps {
                echo 'Deploying the app using docker-compose...'
                bat 'docker-compose up -d'
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
