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
                    sh 'sudo docker images -a'
                    sh 'sudo docker build -t JuiceShop .'
                    sh 'sudo docker images -a'
                }
            }
        }
    }
}
