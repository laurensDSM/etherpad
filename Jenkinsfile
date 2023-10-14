pipeline {
    agent any
    stages {
        stage('Image Cleanup') {
            steps {
                sh 'docker system prune -a --volumes --force'
            }
        }
        stage('Build') {
            steps {
                sh 'docker compose build'
            }
        }
    }
    post {
        failure {
            sh 'echo "Pipeline failed!"'
        }
    }     
}