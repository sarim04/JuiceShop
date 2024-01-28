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
        stage('Test and Build'){
            parallel{
                stage('Snyk-Scan'){
                    steps{
                        script{
                            sh 'npm install'
                            snykSecurity failOnError: false, failOnIssues: false, organisation: 'sarim04', projectName: 'juice-shop', snykInstallation: 'snyk-community', snykTokenId: 'snyk_token', additionalArguments: '--json-file-output=dependecyCheck_results.json'
                            }
                        }    
                    }
                stage('Secret Scanning'){
                       steps{
                            script{
                                sh 'echo "Running Secret Scanning using Trufflehog"'
                                sh 'docker run --rm -i -v "$PWD:/repo" trufflesecurity/trufflehog:latest git file:///repo --no-update --entropy --regex --concurrency=2 --include-detectors="all" --json-legacy >> trufflehog_output.json'
                                sh 'sed -i "/trufflehog/d" trufflehog_output.json'
                            }
                        }
                }
                
                stage('SAST'){
                    steps{
                        script{
                            try {
                                sh 'echo $PWD'
                                sh 'docker run --rm -i -e "SNYK_TOKEN=$SNYK_CREDENTIALS" -v "$PWD:/project" -v "$PWD:/app" snyk/snyk:alpine snyk code test --json-output-file=snykcode_results.json --org=sarim04'
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
                        sh 'docker run --rm -i -e "SNYK_TOKEN=$SNYK_CREDENTIALS" -v "$PWD:/app" snyk/snyk:alpine snyk container test --username=$DOCKERHUB_CREDENTIALS_USR --password=$DOCKERHUB_CREDENTIALS_PSW --json-file-output=container_vuln.json --app-vulns sarim04/juiceshop:latest --org=sarim04'
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
            archiveArtifacts artifacts: "dependecyCheck_results.json"
            archiveArtifacts artifacts: "trufflehog_output.json"
            archiveArtifacts artifacts: "snykcode_results.json"
            archiveArtifacts artifacts: "container_vuln.json" 
        }
    }

}
