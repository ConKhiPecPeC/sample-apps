pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'd316831b-7188-4dd8-96dd-9c3435ca8877' 
        DOCKER_IMAGE = 'conkhipecpec/2048-jenkins'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Hello !'
            }
        }

        stage('Check Out') {
            steps {
                echo 'Check by Sonarqube'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Log in to Docker Hub using credentials and push the Docker image
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                        try {
                            echo "Pushing Docker image ${DOCKER_IMAGE}:${DOCKER_TAG} to Docker Hub..."
                            dockerImage.push("${DOCKER_TAG}")
                            echo "Docker image successfully pushed."
                        } catch (Exception e) {
                            echo "Failed to push Docker image: ${e.getMessage()}"
                            error("Docker push failed.")
                        }
                    }
                }
            }
        }
        
    }
}