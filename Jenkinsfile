pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('github-token')
        IMAGE_NAME = 'laurensdsm/etherpad'
        IMAGE_VERSION = 'latest'
    }
    stages {
        stage('Image Cleanup') {
            steps {
                sh 'docker system prune -a --volumes --force'
            }
        }
        // stage('OWASP Dependency Check') {
        //     steps {
        //             script {
        //                 def additionalArguments = '''\
        //                     -o ./
        //                     -s ./
        //                     -f ALL
        //                     --prettyPrint
        //                 '''
        //                 dependencyCheck(
        //                     additionalArguments: additionalArguments,
        //                     odcInstallation: 'owasp'
        //                 )
        //             }
        //             dependencyCheckPublisher(pattern: 'dependency-check-report.xml')
        //         }
        //     }
        // stage('Build') {
        //     steps {
        //         sh 'docker compose build'
        //     }
        // }
        // stage('Trivy Check') {
        //     steps {
        //         sh "trivy image --no-progress --exit-code 0 --severity MEDIUM,HIGH,CRITICAL --format template --template '@/usr/local/share/trivy/templates/html.tpl' -o trivy_report.html laurensdsm/etherpad:latest"
        //         archiveArtifacts artifacts: 'trivy_report.html', allowEmptyArchive: true
        //     }
        // }
        // stage('Install npm dependencies') {
        //     steps {
        //         script {
        //             dir("${WORKSPACE}") {
        //                 sh 'npm init -y'
        //                 sh 'npm install'
        //             }
        //         }
        //     }
        // }

        // stage ('Trufflehog Check') {
        //     steps {
        //         sh 'docker pull trufflesecurity/trufflehog:latest'
        //         sh 'rm trufflehog_results.json || true'
        //         sh 'rm trufflehog_results.html || true'
        //         sh 'docker run -v "$WORKSPACE:/pwd" trufflesecurity/trufflehog:latest filesystem /pwd --json 2>&1 | grep -v "unable to read file for MIME type detection: EOF" > trufflehog_results.json'
        //         sh './convert_json_to_html.sh'
        //         sh 'rm package-lock.json || true'
        //         sh 'rm package.json || true'
        //         archiveArtifacts artifacts: 'trufflehog_results.html', allowEmptyArchive: true
        //     }
        // }
        stage('ESLint Check') {
            steps {
                script {
                    def eslintError = null 
                    try {
                        sh 'rm -rf node_modules package-lock.json && npm install'
                        sh 'rm eslint.xml || true'
                        sh './node_modules/eslint/bin/eslint.js -f checkstyle src > eslint.xml'
                    } catch (Exception e) {
                        // Catch any exception and handle it gracefully
                        eslintError = "ESLint Check failed: ${e.message}"
                        echo eslintError
                        currentBuild.result = 'SUCCESS' // Set overall build result to SUCCESS
                    }

                    archiveArtifacts artifacts: 'eslint.xml', allowEmptyArchive: true
                }
            }
        }
        // stage('Deployment') {
        //     steps {
        //         sh 'docker compose up -d'
        //     }
        // }
        // stage('DAST SCAN') {
        //     steps {
        //         sh 'rm nuclei.txt || true'
        //         sh 'rm nuclei_json.txt || true'
        //         sh 'rm nuclei1.html || true'
        //         sh 'rm nuclei2.html || true'
        //         sh 'echo "DAST SCAN"'
        //         sh 'docker pull projectdiscovery/nuclei:latest'
        //         sh 'docker run --rm projectdiscovery/nuclei:latest -u http://192.168.0.141:9001/ --no-color > nuclei.txt'
        //         sh 'docker run --rm projectdiscovery/nuclei:latest -u http://192.168.0.141:9001/ --no-color -j  > nuclei_json.txt'
        //         sh './convert_nuclei.sh'
        //         sh './convert_nuclei_json.sh'
        //         archiveArtifacts artifacts: 'nuclei1.html', allowEmptyArchive: true
        //         archiveArtifacts artifacts: 'nuclei2.html', allowEmptyArchive: true
        //     }
        // }
        // stage('Login to GHCR') {
        //     steps {
        //         sh 'echo $GITHUB_TOKEN_PSW | docker login ghcr.io -u $GITHUB_TOKEN_USR --password-stdin'
        //     }
        // }

        // stage('Tag Image') {
        //     steps {
        //         sh "docker tag $IMAGE_NAME:$IMAGE_VERSION ghcr.io/$IMAGE_NAME:$IMAGE_VERSION"
        //     }
        // }

        // stage('Push Image') {
        //     steps {
        //         sh "docker push ghcr.io/$IMAGE_NAME:$IMAGE_VERSION"
        //     }
        // }
        
        // stage ('Notation Login') {
        //     steps {
        //         sh 'echo $GITHUB_TOKEN_PSW | notation login ghcr.io -u $GITHUB_TOKEN_USR --password-stdin'
        //     }
        // }

        // stage ('Image Signing') {
        //     steps {
        //         sh 'if [ ! -f /var/lib/jenkins/.config/notation/localkeys/etherpad.org.key ]; then notation cert generate-test --default "etherpad.org"; fi'
        //         script {
        //             def signedImage = sh(script: 'notation sign ghcr.io/$IMAGE_NAME:$IMAGE_VERSION', returnStdout: true).trim()
        //             echo signedImage
        //             def indexOfAt = signedImage.lastIndexOf('@')
        //             if (indexOfAt >= 0) {
        //                 env.SIGNED_IMAGE = 'ghcr.io/' + IMAGE_NAME + ':' + IMAGE_VERSION + signedImage.substring(indexOfAt)
        //             } else {
        //                 currentBuild.result = 'FAILURE'
        //                 error("Invalid signed image format: $signedImage")
        //             }                                    
        //         }    
        //     }
        // }
        // stage ('Image Verification') {
        //     steps {
        //         sh 'notation policy import ./trustpolicy.json'
        //         sh 'notation verify $SIGNED_IMAGE'
        //     }
        // }
    }
    post {
        failure {
            sh 'echo "Pipeline failed!"'
        }
        always{
            sh 'docker logout'
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
            publishHTML(
                target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: false,
                    reportDir: '.',
                    reportFiles: 'nuclei1.html',
                    reportName: 'Nuclei Basic ',
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
                    reportFiles: 'nuclei2.html',
                    reportName: 'Nuclei Extend ',
                    reportTitles: '',
                    useWrapperFileDirectly: true
                ]
            )
            recordIssues enabledForFailure: true, aggregatingResults: true, tool: checkStyle(pattern: 'eslint.xml')
        }
    }     
}
