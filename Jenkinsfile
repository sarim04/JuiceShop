pipeline {
    agent any
    options { 
        skipDefaultCheckout() 
    }
    stage('Clone Repo'){
            steps{
                script{
                    sh 'rm -rf JuiceShop'
                    sh 'git clone https://github.com/sarim04/JuiceShop.git'
                }
            }
        }
    stages {
        stage('Secret Scanning'){
            steps{
                script{
                    sh 'set +x'
                    sh 'trufflehog git file://../testStep --no-update --entropy --regex --concurrency=2 --include-detectors="all" --json-legacy > trufflehog_results.json' 
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
