pipeline {
    agent any
    stages {
        stage('Verify Branch'){
            steps{
            echo "$GIT_BRANCH"
            }
        }
        stage('PWD'){
            steps{
                script{
                    sh 'pwd'
                    sh 'ls -al'
                }
            }
        }
        stage('Build'){
            steps{
                script{
                    sh 'docker images -a'
                    sh 'docker build -t JuiceShop .'
                    sh 'docker images -a'
                }
            }
        }
    }
}
