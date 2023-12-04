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
    }
}
