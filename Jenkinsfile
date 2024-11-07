pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'conkhipecpec/2048-jenkins'
        DOCKER_CONFIG = "${env.WORKSPACE}/.docker" // Set a writable Docker config path
        SONAR_TOKEN = credentials('sonarcloud-token')
        GOOGLE_CLOUD_SSH = 'google-cloud-ssh'
        DOCKER_TAG = "latest"
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
                        def qualityGate = waitForQualityGate() // Requires SonarQube plugin in Jenkins
                        if (qualityGate.status != 'OK') {
                            error "Pipeline aborted due to Quality Gate failure: ${qualityGate.status}"
                        }
                    }
                }
            }
        }
/* 
        stage('Build Docker Image'){
            steps{
                sh '''
                    if docker image inspect ${DOCKER_IMAGE}:${DOCKER_TAG} > /dev/null 2>&1; then
                        docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}
                    fi
                '''
                sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }

        stage('Push Image to Docker Hub'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'Docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    sh 'docker push ${DOCKER_IMAGE}:${DOCKER_TAG}'
                }
            }
        }
*/
        stage('Pull Docker Image and Run Container') {
            steps {
                script {
                    // Using the SSH Publisher to execute commands on the Google Cloud instance
                    sshPublisher(publishers: [
                        sshPublisherDesc(
                            configName: GOOGLE_CLOUD_SSH,
                            transfers: [
                                sshTransfer(
                                    sourceFiles: '',  // No files to transfer
                                    execCommand: """
                                        # Ensure Docker is installed and running
                                        if ! command -v docker &> /dev/null
                                        then
                                            echo "Docker not found, installing Docker..."
                                            curl -fsSL https://get.docker.com | bash
                                            sudo systemctl start docker
                                            sudo systemctl enable docker
                                        fi

                                        # Check Docker service status
                                        echo "Checking Docker service status..."
                                        sudo systemctl status docker || sudo systemctl start docker

                                        # Pull the Docker image
                                        echo "Pulling Docker image ${DOCKER_IMAGE}..."

                                        docker pull ${DOCKER_IMAGE}:${DOCKER_TAG} &&
                                        docker stop 2048-game || true &&
                                        docker rm 2048-game || true &&
                                        sudo docker run -d -it --name 2048-Game -p 80:8080 ${DOCKER_IMAGE}:${DOCKER_TAG}
                                        
                                        # Print Docker logs if something goes wrong
                                        echo "Docker logs:"
                                        sudo journalctl -u docker.service --since "1 hour ago" || echo "No Docker logs found"
                                    """,
                                    remoteDirectory: '',  // No file upload, only command execution
                                    execTimeout: 600000  // Allowing 10 minutes for the commands to complete
                                )
                            ]
                        )
                    ])
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