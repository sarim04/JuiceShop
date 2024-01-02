pipeline {
    agent any
    environment {
    DOCKERHUB_CREDENTIALS = credentials('DockerHub_PAT')
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
                            snykSecurity failOnError: false, failOnIssues: false, organisation: 'sarim04', projectName: 'juice-shop', snykInstallation: 'Snyk-Community', snykTokenId: 'Snyk_Auth_Token'
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
                            }
                        }
                }
                
                stage('SAST'){
                    steps{
                        script{
                            sh 'echo "Running SAST Scan using Snyk Code"'
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

}
