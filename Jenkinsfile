pipeline {
    agent any
    tools {
     jdk 'JDK8'
    }
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "deepakchp/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                 sh 'sudo kubectl apply -f train-schedule-kube-canary.yml'
            }
        }
        stage('DeployToProduction') {
            
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                sh 'sudo kubectl apply -f train-schedule-kube-canary.yml train-schedule-kube.yml'                
            }
        }
    }
}
