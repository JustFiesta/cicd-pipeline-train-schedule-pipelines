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
    }
}
