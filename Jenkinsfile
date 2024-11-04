pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'conkhipecpec/2048-jenkins'
        DOCKER_CONFIG = "${env.WORKSPACE}/.docker" // Set a writable Docker config path
        SONAR_TOKEN = credentials('sonarcloud-token')
        SONAR_HOST_URL = credentials('SONAR_HOST_URL')
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
            def scannerHome = tool 'sonar-scanner';
            steps {
                withSonarQubeEnv('SonarCloud') {  // Uses SonarQube environment configuration in Jenkins
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage("build") {
            //agent { node {label 'master'}}
            environment {
                DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${GIT_COMMIT.substring(0,7)}"
            }
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . "
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                sh "docker image ls | grep ${DOCKER_IMAGE}"
                withCredentials([usernamePassword(credentialsId: 'Docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }

                //clean to save disk
                sh "docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh "docker image rm ${DOCKER_IMAGE}:latest"
            }
        }
        
    }

    post{
        success{
            echo "SUCCESSFUL"
        }
        failure{
            echo "FAILED"
        }
    }
    
}