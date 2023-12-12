pipeline {
    agent any
    stages {
        stage('Verify Branch'){
            steps{
            echo "$GIT_BRANCH"
            }
        }
        stage('Build and Static-Testing'){
            parallel{
                stage('Snyk-Scan'){
                    steps{
                        script{
                            snykSecurity failOnError: false, failOnIssues: false, organisation: 'sarim04', projectName: 'juice-shop', snykInstallation: 'Snyk-Community', snykTokenId: 'snyk-token'
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
                            sh 'sudo docker images -a'
                            sh 'sudo docker build -t juiceshop .'
                            sh 'sudo docker images -a'
                            }
                        }
                    }
                }
        }

    }
}
