pipeline {
    agent any
    tools {
        gradle '5.1'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
                sh 'gradle --version'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
    }
}
