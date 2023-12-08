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
                        docker{
                            image "docker.io/hadolint/hadolint:v1.18.0"
                            reuseNode true 
                        }
                    }
                    script{
                        sh label: "Lint Dockerfile", script: "hadolint Dockerfile > hadolint-results.txt"
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
