pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "conkhipecpec/2048-jekins"
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Home Page !'
            }
        }
        stage('Run subfolder Jenkinsfile'){
            steps{
                script {
                    // Load and execute Jenkinsfile from the subfolder
                    def subPipeline = load 'game-2048-example/Jenkinsfile'
                    subPipeline.run()
                }
            }
        }

    }
}