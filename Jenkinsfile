pipeline {
    agent {
        label 'test-server'
    }
    options { 
        skipDefaultCheckout() 
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('DockerHub_PAT')
        SNYK_CREDENTIALS = credentials('Snyk_Auth_Token')
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
                    sh 'docker run --rm -i -v "$PWD:/repo" trufflesecurity/trufflehog:latest git file:///repo/testStep --no-update --entropy --regex --concurrency=2 --include-detectors="all" --json-legacy > trufflehog_results.json' 
                    sh 'cat trufflehog_results.json'
                }
            }
        }
        stage('Test and Build'){
            parallel{
                stage('Snyk-Scan'){
                    steps{
                        script{
                            sh 'npm install'
                            snykSecurity failOnError: false, failOnIssues: false, monitorProjectOnBuild: false, organisation: 'sarim04', projectName: 'juice-shop', snykInstallation: 'snyk-community', snykTokenId: 'snyk_token', additionalArguments: '--sarif-file-output=dependencyCheck_results.sarif'
                            }
                        }    
                    }                
                stage('SAST'){
                    steps{
                        script{
                            try {
                                sh 'echo $PWD'
                                sh 'docker run --rm -i -e "SNYK_TOKEN=$SNYK_CREDENTIALS" -v "$PWD:/project" -v "$PWD/testStep:/app" snyk/snyk:alpine snyk code test --sarif-file-output=snykCode_results.sarif --org=sarim04'
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
                        sh 'docker run --rm -i -e "SNYK_TOKEN=$SNYK_CREDENTIALS" -v "$PWD/testStep:/app" snyk/snyk:alpine snyk container test --username=$DOCKERHUB_CREDENTIALS_USR --password=$DOCKERHUB_CREDENTIALS_PSW --sarif-file-output=snykContainer_results.sarif --app-vulns sarim04/juiceshop:latest --org=sarim04'
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
            archiveArtifacts artifacts: "dependencyCheck_results.sarif"
            archiveArtifacts artifacts: "trufflehog_results.json"
            archiveArtifacts artifacts: "JuiceShop/snykCode_results.sarif"
            archiveArtifacts artifacts: "JuiceShop/snykContainer_results.sarif" 
        }
    }

}
