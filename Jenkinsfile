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
        stage('Trivy Check') {
            steps {
                sh "trivy image --no-progress --exit-code 0 --severity MEDIUM,HIGH,CRITICAL --format template --template '@/usr/local/share/trivy/templates/html.tpl' -o trivy_report.html laurensdsm/etherpad-lite:latest"
                archiveArtifacts artifacts: 'trivy_report.html', allowEmptyArchive: true
            }
        }
        stage('Install npm dependencies') {
            steps {
                script {
                    dir("${WORKSPACE}") {
                        sh 'npm init -y'
                        sh 'npm install'
                    }
                }
            }
        }

        stage ('Trufflehog Check') {
            steps {
                sh 'docker pull trufflesecurity/trufflehog:latest'
                sh 'rm trufflehog_results.json || true'
                sh 'rm trufflehog_results.html || true'
                sh 'docker run -v "$WORKSPACE:/pwd" trufflesecurity/trufflehog:latest filesystem /pwd --json 2>&1 | grep -v "unable to read file for MIME type detection: EOF" > trufflehog_results.json'
                sh './convert_json_to_html.sh'
                sh 'rm package-lock.json || true'
                sh 'rm package.json || true'
                archiveArtifacts artifacts: 'trufflehog_results.html', allowEmptyArchive: true
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
        always{
            publishHTML(
                target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: false,
                    reportDir: '.',
                    reportFiles: 'trivy_report.html',
                    reportName: 'Trivy Report',
                    reportTitles: '',
                    useWrapperFileDirectly: true
                ]
            )
            publishHTML(
                target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: false,
                    reportDir: '.',
                    reportFiles: 'trufflehog_results.html',
                    reportName: 'Trufflehog Report',
                    reportTitles: '',
                    useWrapperFileDirectly: true
                ]
            )
        }
    }     
}