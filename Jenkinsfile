pipeline {
    agent any
    options { 
        skipDefaultCheckout() 
    }
    stages {
        stage('Clone Repo'){
            steps{
                script{
                    checkout scm
                    sh 'ls'
                }
            }
        }
        stage('Secret Scanning'){
            steps{
                script{
                    sh 'set +x'
                    sh 'trufflehog git file://JuiceShop --no-update --entropy --regex --concurrency=2 --include-detectors="all" --json-legacy > trufflehog_results.json' 
                    sh 'cat trufflehog_results.json'
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: "trufflehog_results.json"
        }
    }

}
