pipeline {
    agent any
    environment {
        // Define environment variables
        // Jenkins credentials configuration
        DOCKER_HUB_CREDENTIALS = 'dockerhub_credentials' // Docker Hub credentials ID stored in Jenkins
        // Docker Hub Repository's name
        DOCKER_IMAGE = 'yifans0214/teedy' // Your Docker Hub user name and Repository's name
        DOCKER_TAG = "${env.BUILD_NUMBER}" // Use build number as tag
    }
    stages {
        stage('Build') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/whiteki08/Teedy']] // Your GitHub Repository
                )
                sh 'mvn --version' // Verify Maven is available
                sh 'mvn -B -DskipTests clean package'
            }
        }
        
        // Building Docker images
        stage('Building image') {
            steps {
                script {
                    // Assume Dockerfile located at root
                    docker.build("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}")
                }
            }
        }
        
        // Uploading Docker images into Docker Hub
        stage('Upload image') {
            steps {
                script {
                    // Sign in Docker Hub
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_HUB_CREDENTIALS) {
                        // Push image
                        docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").push()
                        // Optional: label latest
                        docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").push('latest')
                    }
                }
            }
        }
    }
}