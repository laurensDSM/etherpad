pipeline {
    agent any
    stages {
        stage('Image Cleanup') {
            steps {
                sh 'docker system prune -a --volumes --force'
            }
        }
	    stage('OWASP Dependency Check') {
            steps {
                    script {
                        def additionalArguments = '''\
                            -o ./
                            -s ./
                            -f ALL
                            --prettyPrint
                        '''

                        dependencyCheck(
                            additionalArguments: additionalArguments,
                            odcInstallation: 'owasp'
                        )
                    }

                    dependencyCheckPublisher(pattern: 'dependency-check-report.xml')
                }
            }
        stage('Build') {
            steps {
                sh 'docker compose build'
            }
        }
        stage('Deployment') {
            steps {
                sh 'docker compose up -d'
            }
        }
    }
    post {
        failure {
            sh 'echo "Pipeline failed!"'
        }
    }     
}