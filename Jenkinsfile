pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'conkhipecpec/2048-jenkins'
        DOCKER_CONFIG = "${env.WORKSPACE}/.docker" // Set a writable Docker config path
        SONAR_TOKEN = credentials('sonarcloud-token')
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Hello !'
            }
        }

        stage('Check Out') {
            steps {
                checkout scm
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                script {
                    // Define the scanner home
                    scannerHome = tool 'sonar-scanner'
                }

                withSonarQubeEnv('SonarCloud') {
                    // Run the SonarScanner with the required properties
                    sh """
                    ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey="ConKhiPecPeC_sample-apps" \
                        -Dsonar.organization="conkhipecpec" \
                        -Dsonar.host.url="https://sonarcloud.io" \
                        -Dsonar.login=\$SONAR_TOKEN
                    """
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') { // Wait up to 5 minutes for Quality Gate result
                    script {
                        def qualityGate = waitForQualityGate() 
                        if (qualityGate.status != 'OK') {
                            error "Pipeline aborted due to Quality Gate failure: ${qualityGate.status}"
                        }
                    }
                }
            }
        }

        stage("Build And Push Docker Image") {
            environment {
                DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${GIT_COMMIT.substring(0, 7)}"
            }
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                sh "docker image ls | grep ${DOCKER_IMAGE}"
                withCredentials([usernamePassword(credentialsId: 'Docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }

                // Clean up to save disk space
                sh "docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh "docker image rm ${DOCKER_IMAGE}:latest"
            }
        }

        stage('Deploy to Google Cloud') {
            environment {
                GOOGLE_CLOUD_IP = '34.57.18.241'
                SSH_KEY = credentials('google-cloud-ssh-key') // Set up the private key in Jenkins credentials
            }
            steps {
                script {
                    // Use SSH to connect to the Google Cloud instance and deploy the Docker container
                    sh """
                    ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} user@${GOOGLE_CLOUD_IP} << EOF
                        docker login --username=$DOCKER_USERNAME --password=$DOCKER_PASSWORD
                        docker pull ${DOCKER_IMAGE}:latest
                        docker stop my-app || true  # Stop any running container named my-app
                        docker rm my-app || true    # Remove the stopped container if it exists
                        docker run -d --name my-app -p 80:80 ${DOCKER_IMAGE}:latest
                    EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo "SUCCESSFUL"
        }
        failure {
            echo "FAILED"
        }
    }
}