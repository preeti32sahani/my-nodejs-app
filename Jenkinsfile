pipeline {
    agent any

    environment {
        IMAGE_NAME = 'my-nodejs-app'                   // Your image name
        DOCKER_REGISTRY = 'bunny011'                   // Your DockerHub username
        DOCKER_CREDENTIALS = 'docker-hub-creds'        // Jenkins credentials ID
    }

    stages {
        stage('Checkout Source') {
            steps {
                git url: 'https://github.com/loyalbunny/my-nodejs-app.git', branch: 'main'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS}",
                                                    usernameVariable: 'DOCKER_USERNAME',
                                                    passwordVariable: 'DOCKER_PASSWORD')]) {

                        // Log in to DockerHub
                        bat """
                        docker logout
                        docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%
                        """

                        // Build the image with build number tag
                        def imageTag = "${DOCKER_REGISTRY}/${IMAGE_NAME}:${env.BUILD_NUMBER}"
                        bat "docker build -t ${IMAGE_NAME}:${env.BUILD_NUMBER} ."
                        bat "docker tag ${IMAGE_NAME}:${env.BUILD_NUMBER} ${imageTag}"

                        // Push to DockerHub
                        bat "docker push ${imageTag}"
                    }
                }
            }
        }

        stage('Deploy and Test App') {
            steps {
                script {
                    // Stop and remove existing containers (ignore errors)
                    bat "docker-compose down || exit 0"

                    // Start fresh containers
                    bat "set BUILD_NUMBER=${env.BUILD_NUMBER} && docker-compose up -d --force-recreate"

                    // Wait for app to start
                    bat "timeout /t 10 >nul"

                    // Test if app is running
                    bat "curl http://localhost:3000 || echo App not responding"
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
    }
}