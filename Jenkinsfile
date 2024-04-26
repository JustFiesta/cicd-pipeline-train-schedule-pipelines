pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "testfiesta/train-schedule"
    }
    tools {
        gradle '7.3'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
                sh 'gradle wrapper --gradle-version=7.3.2'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'canary-deployment'
            }
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
            when {
                branch 'canary-deployment'
            }
            steps {
                script {
                    // set registry with jenkins credentials
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        // reference app var from Build Docker Image stage
                        app.push("${env.BUILD_NUMBER}") // pushing both number tag and latest on same build
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
