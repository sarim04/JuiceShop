pipeline {
    agent any
    stages {
        stage('Secret Scanning'){
            steps{
                script{
                    sh 'set +x'
                    sh 'trufflehog git file://. --no-update --entropy --regex --concurrency=2 --include-detectors="all" --json-legacy > trufflehog_results.json 1>&2' 
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
