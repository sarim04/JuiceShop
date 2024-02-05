pipeline {
    agent any
    options {
        skipDefaultCheckout()
    }
    stages {
        stage('checkout'){
            steps{
                checkout scm
            }
        }
        stage('pwd'){
            steps{
                script{
                    sh 'pwd'
                    sh 'ls'
                }
            }
        }
        stage('Secret Scanning'){
            steps{
                script{
                    sh 'set +x'
                    sh 'trufflehog git file://. --no-update --entropy --regex --concurrency=2 --include-detectors="all" --json-legacy > trufflehog_results.json 2>&1' 
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
