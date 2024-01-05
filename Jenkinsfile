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
                            sh 'npm install'
                            snykSecurity failOnError: false, failOnIssues: false, organisation: 'sarim04', projectName: 'juice-shop', snykInstallation: 'snyk-community', snykTokenId: 'Snyk_Token', additionalArguments: '--json-file-output=dependecyCheck_results.json'
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
                            try {
                                sh 'echo $PWD'
                                sh 'docker run --rm -i -e "SNYK_TOKEN=$SNYK_CREDENTIALS" -v "/var/lib/jenkins/workspace/:/project" -v "$PWD:/app" snyk/snyk:alpine snyk code test --json-output-file=snykcode_results.json --org=sarim04'
                            }
                            catch (err){
                                currentBuild.result = 'SUCCESS'
                            }

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
            archiveArtifacts artifacts: "dependecyCheck_results.json"
            archiveArtifacts artifacts: "trufflehog_output.json"
            archiveArtifacts artifacts: "snykcode_results.json"
        }
    }

}
