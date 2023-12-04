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
                sh label: "Print CWD"
                script: "pwd"
            }
        }
    }
}
