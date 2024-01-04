pipeline {
    agent any
    environment {
    DOCKERHUB_CREDENTIALS = credentials('DockerHub_PAT')
    SNYK_CREDENTIALS = credentials('Snyk_Auth_Token')
      }
    stages {
        stage('Verify Branch'){
            steps{
            echo "$GIT_BRANCH"
            }
        }
        stage('Test and Build'){
            parallel{
                stage('Snyk-Scan'){
                    steps{
                        script{
                            snykSecurity failOnError: false, failOnIssues: false, organisation: 'sarim04', projectName: 'juice-shop', snykInstallation: 'Snyk-Community', snykTokenId: 'Snyk_Auth_Token', additionalArguments: '--json-file-output=dependecyCheck_results.json'
                            }
                        }    
                    }
                stage('Lint'){                    
                    steps{
                        script{
                            sh 'echo "Linting the code with Hadolint"'
                            }
                        }
                }
                
                stage('Secret Scanning'){
                       steps{
                            script{
                                sh 'echo "Running Secret Scanning using Trufflehog"'
                                sh 'trufflehog git file://. --regex --include-detectors="all" --no-update --json-legacy >> trufflehog_output.json'
                            }
                        }
                }
                
                stage('SAST'){
                    steps{
                        script{
                            sh 'echo $PWD'
                            sh 'docker run --rm -it -e "SNYK_TOKEN=$SNYK_CREDENTIALS_PSW" -v "../:/project" -v "$PWD:/app" snyk/snyk:alpine snyk code test --json --org=sarim04 >> snykcode_results.json'
                            }
                        }
                }
                stage('Build'){
                    steps{
                        script{
                            sh 'docker images -a'
                            sh 'docker build -t sarim04/juiceshop .'
                            sh 'docker images -a'
                            }
                        }
                    }
                }
        }
    stage('Push'){
        steps{
            script{
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push sarim04/juiceshop'
                    }
                }
            }
    }
    post {
        always {
            archiveArtifacts artifacts: "trufflehog_output.json"
            archiveArtifacts artifacts: "snykcode_results.json"
        }
    }

}
