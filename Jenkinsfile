pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'conkhipecpec/2048-jenkins'
        DOCKER_CONFIG = "${env.WORKSPACE}/.docker" // Set a writable Docker config path
        SONAR_TOKEN = credentials('sonarcloud-token')
        GOOGLE_CLOUD_SSH = 'google-cloud-ssh'
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
            environment{
                DOCKER_TAG = "lastest"
            }
            steps{
                sh 'docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}
                sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }
*/
        stage('Push Image to Docker Hub'){
            environment{
                DOCKER_TAG = "lastest"
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'Docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    sh 'docker push ${DOCKER_IMAGE}:${DOCKER_TAG}'
                }
            }
        }
/* 
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
                                        sudo docker pull ${DOCKER_IMAGE}

                                        # Run the Docker container with --rm to avoid name conflicts
                                        echo "Running Docker container from image ${DOCKER_IMAGE}..."
                                        sudo docker run -d --rm -it -p 8080:8080 --name my-container ${DOCKER_IMAGE}

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
        */

        
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