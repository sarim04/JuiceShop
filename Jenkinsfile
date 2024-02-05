pipeline {
    agent any
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
                checkout scm
            }
        }
        stage('Test and Build'){
            parallel{
                stage('Secret Scanning'){
                    steps{
                        script{
                            sh 'set +x'
                            sh 'trufflehog git file://. --no-update --entropy --regex --concurrency=2 --include-detectors="all" --json-legacy > trufflehog_results.json 2>&1' 
                            sh 'cat trufflehog_results.json'
                            }
                        }
                    }
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
                                sh 'docker run --rm -i -e "SNYK_TOKEN=$SNYK_CREDENTIALS" -v "$PWD:/project" -v "$PWD:/app" snyk/snyk:alpine snyk code test --sarif-file-output=snykCode_results.sarif --org=sarim04'
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
                        sh 'docker run --rm -i -e "SNYK_TOKEN=$SNYK_CREDENTIALS" -v "$PWD/JuiceShop:/app" snyk/snyk:alpine snyk container test --username=$DOCKERHUB_CREDENTIALS_USR --password=$DOCKERHUB_CREDENTIALS_PSW --sarif-file-output=snykContainer_results.sarif --app-vulns sarim04/juiceshop:latest --org=sarim04'
                    }
                    catch (err){
                        currentBuild.result = 'SUCCESS'
                    }
                }
            }
        }
        stage('deploy'){
            agent {
                label 'test-server'
            }
            steps{
                script{
                    sh 'docker run -d --network host sarim04/juiceshop:latest'
                }
            }
        }
        stage('dast scan'){
            steps{
                script{
                    try{
                        sh 'docker run -v "$(pwd):/zap/wrk/:rw" --user root -i ghcr.io/zaproxy/zaproxy:stable zap-full-scan.py -t http://192.168.15.139:3000/ -x zap_results.xml'
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
            archiveArtifacts artifacts: "snykCode_results.sarif"
            archiveArtifacts artifacts: "snykContainer_results.sarif"
         }
    }
}
