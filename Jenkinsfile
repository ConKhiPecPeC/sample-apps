pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'conkhipecpec/2048-jenkins'
        DOCKER_CONFIG = "${env.WORKSPACE}/.docker" 
        SONAR_TOKEN = credentials('sonarcloud-token')
        GOOGLE_CLOUD_IP = credentials('google_cloud_ip')  
        SSH_KEY = credentials('google-cloud-ssh-key')
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

        stage('Connect to Google Cloud and Deploy Docker Container') {
            steps {
                script {
                    // Run SSH command to pull Docker image and start container
                    sh """
                    ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} huy123@${GOOGLE_CLOUD_IP} << 'EOF'
                        echo "Logging into Docker Hub..."
                        echo \$DOCKER_PASSWORD | docker login --username \$DOCKER_USERNAME --password-stdin || exit 1
                        
                        echo "Pulling latest Docker image..."
                        docker pull ${DOCKER_IMAGE}:latest || exit 1
                        
                        echo "Stopping any existing container named 'my-app'..."
                        docker stop my-app || true
                        docker rm my-app || true
                        
                        echo "Running new Docker container..."
                        docker run -d --name my-app -p 80:80 ${DOCKER_IMAGE}:latest || exit 1
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