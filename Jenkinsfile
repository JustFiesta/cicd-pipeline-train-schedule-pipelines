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
        stage('DeployToProduction') {
            when {
                branch 'canary-deployment'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'prod', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker pull testfiesta/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker stop train-schedule\""
                            sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker rm train-schedule\""
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"docker run --restart always --name train-schedule -p 8080:8080 -d testfiesta/train-schedule:${env.BUILD_NUMBER}\""
                    }
                }
            }
        }
    }
}
