pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        //added comment
        DOCKER_IMAGE_NAME = "prashanth87/train-schedule"
    }
    stages {
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
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
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
                withKubeConfig([credentialsId: 'kubeconfigId']) {
                    sh 'envsubst <train-schedule-kube-canary.yml | kubectl apply -f -'
                }
            }
        }
        stage('DeployToProduction') {
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                milestone(1)
                withKubeConfig([credentialsId: 'kubeconfigId']) {
                    sh 'envsubst <train-schedule-kube-canary.yml | kubectl apply -f -'
                }
                withKubeConfig([credentialsId: 'kubeconfigId']) {
                    sh 'envsubst <train-schedule-kube.yml | kubectl apply -f -'
                }
            }
        }
    }
}
