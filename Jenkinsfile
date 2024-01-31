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
        stage('Secret Scanning'){
            steps{
                script{
                    sh 'echo "Running Secret Scanning using Trufflehog"'
                    sh 'set +x'
                    sh 'docker run --rm -i -v "$PWD:/repo" trufflesecurity/trufflehog:latest git file:///repo/ --no-update --entropy --regex --concurrency=2 --include-detectors="all" --json-legacy > trufflehog_results.json' 
                        }
                    }
                }
        stage('Test and Build'){
            parallel{
                stage('Snyk-Scan'){
                    steps{
                        script{
                            sh 'npm install'
                            snykSecurity failOnError: false, failOnIssues: false, monitorProjectOnBuild: false, organisation: 'sarim04', projectName: 'juice-shop', snykInstallation: 'snyk-community', snykTokenId: 'snyk_token', additionalArguments: '--json-file-output=dependencyCheck_results.json'
                            }
                        }    
                    }                
                stage('SAST'){
                    steps{
                        script{
                            try {
                                sh 'echo $PWD'
                                sh 'docker run --rm -i -e "SNYK_TOKEN=$SNYK_CREDENTIALS" -v "$PWD:/project" -v "$PWD:/app" snyk/snyk:alpine snyk code test --json-file-output=snykCode_results.json --org=sarim04'
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
                sh 'docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW'
                sh 'docker push sarim04/juiceshop'
                    }
                }
            }
    stage('Container Scan'){
        steps{
            script{
                try {
                        sh 'cd /var/lib/jenkins/workspace/'
                        sh 'docker run --rm -i -e "SNYK_TOKEN=$SNYK_CREDENTIALS" -v "$PWD:/app" snyk/snyk:alpine snyk container test --username=$DOCKERHUB_CREDENTIALS_USR --password=$DOCKERHUB_CREDENTIALS_PSW --json-file-output=snykContainer_results.json --app-vulns sarim04/juiceshop:latest --org=sarim04'
                }
                catch (err){
                    currentBuild.result = 'SUCCESS'
                    }
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: "dependencyCheck_results.json"
            archiveArtifacts artifacts: "trufflehog_results.json"
            archiveArtifacts artifacts: "snykCode_results.json"
            archiveArtifacts artifacts: "snykContainer_results.json" 
        }
    }

}
