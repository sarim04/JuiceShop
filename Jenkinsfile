pipeline {
    agent any
    stages {
        stage('Verify Branch'){
            steps{
            echo "$GIT_BRANCH"
            }
        }
        stage('Static-Testing'){
            parallel{
                stage('Snyk-Scan'){
                    steps{
                        script{
                            snykSecurity failOnError: false, failOnIssues: false, organisation: 'sarim04', projectName: 'juice-shop', snykInstallation: 'Snyk-Community', snykTokenId: 'snyk-token'
                            }
                        }    
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
