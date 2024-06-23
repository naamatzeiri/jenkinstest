pipeline {
    agent {
        kubernetes {
            yamlFile 'build-pod.yaml'
        }
    }

    environment {
        // Define environment variables for Docker credentials
        DOCKER_CREDENTIALS_ID = 'dockerhub' // ID of the Docker credentials in Jenkins
        DOCKER_REGISTRY = 'docker.io' // e.g., docker.io or your private registry
        DOCKER_IMAGE = 'naamatzeiri/jenkinstest' // Replace with your Docker username and image name
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the source code from the main branch of the Git repository
                git branch: 'main', url: 'https://github.com/naamatzeiri/jenkinstest'
            }
        }
        stage('Build Docker Image') {
            steps {
                container('ez-docker-helm-build') { // Assuming the container that can run Docker is named 'docker'
                    script {
                        // Build the Docker image with the latest tag
                        sh "docker build -t ${env.DOCKER_IMAGE}:latest ."
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                container('ez-docker-helm-build') { // Assuming the container that can run Docker is named 'docker'
                    script {
                        // Push the Docker image to the registry
                        docker.withRegistry("https://${env.DOCKER_REGISTRY}", "${env.DOCKER_CREDENTIALS_ID}") {
                            sh "docker push ${env.DOCKER_IMAGE}:latest"
                        }
                    }
                }
            }
        }
    }


    post {
        always {
            container('ez-docker-helm-build') {
            // Clean up any leftover Docker images on the Jenkins agent
            sh 'docker system prune -f'
            }
        }
        success {
            // Notify the user of the successful build
            echo 'Docker image built and pushed successfully.'
        }
        failure {
            // Notify the user of the failed build
            echo 'Build failed.'
        }
    }
}
