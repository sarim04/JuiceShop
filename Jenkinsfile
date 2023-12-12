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
                    agent{
                        script{
                            sh label: "Lint Dockerfile", script: "echo "Linting Code""
                    }
                }
                
                stage('Lint'){
                        script{
                            sh 'echo "Using Hadolint"'
                    }
                }
                
                stage('Secret Scanning'){
                        script{
                            sh 'echo "Scanning for Secrets using Trufflehog"'
                    }
                }
                
                stage('SAST'){
                        script{
                            sh 'echo "Running SAST scans using Snyk Code"'
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
