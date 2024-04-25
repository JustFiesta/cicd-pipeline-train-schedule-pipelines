pipeline {
    agent any
    tools {
        gradle '5.1'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
                sh 'gradle wrapper --gradle-version=5.1'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("testfiesta/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)' // smoke test - checking if docker image works (this sh is inside docker container)
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
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
    }
}
