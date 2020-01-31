#!groovy

pipeline {
    agent {
        docker {
            image 'node:6-alpine'
            args '-p 3000:3000'
        }
    }
    environment {
        CI = 'true' 
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'apk add yarn'
            }
        }
        stage('Test') { 
            steps {
                sh 'echo "Sucesso!!!"' 
            }
        }
    }
}