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
        stage('Install Packages') {
            steps {
                script {
                    try {
                        sh "node -v"
                        sh "npm install -g yarn"
                        sh 'yarn install'
                    } catch (Exception e) {
                //  slackSend (color: 'error', message: "[ FALHA ] Não foi possivel instalar as dependências - ${BUILD_URL}", tokenCredentialId: 'slack-token')
                        sh "echo $e"
                        currentBuild.result = 'ABORTED'
                        error('Erro')
            }
        }
            
      }

/*      stage('Test and Build') {
        steps {
          script {
            try {
              parallel {
                stage('Run Tests') {
                  steps {
                    sh 'yarn run test'
                  }
                }
              }
            } catch (Exception e) {
              slackSend (color: 'error', message: "[ FALHA ] Testes com erro. Revisar a aplicação!\n ${BUILD_URL}", tokenCredentialId: 'slack-token')
              sh "echo $e"
              currentBuild.result = 'ABORTED'
              error('Erro')
            }
          }
        }
      }*/

    stage('Create Build Artifacts') {
      steps {
        script {
          try {
            sh 'yarn run build'
          } catch (Exception e) {
          //  slackSend (color: 'error', message: "[ FALHA ] Erro no build da aplicação!\n ${BUILD_URL}", tokenCredentialId: 'slack-token')
            sh "echo $e"
            currentBuild.result = 'ABORTED'
            error('Erro')
          }
        }
      }
    }

    stage('Deployment') {
      parallel {
        stage('Develop') {
          when {
           branch 'develop'
          }
          steps {
            withAWS(region:'us-east-1',credentials:'s3-conquista') {
              s3Delete(bucket: 'conquista-frontend-dev', path:'**/*')
              s3Upload(bucket: 'conquista-frontend-dev', workingDir:'build', includePathPattern:'**/*', acl:'PublicRead');
            }
            //  slackSend (color: 'good', message: "[ SUCESSO ] Deploy aplicado com sucesso! \n\n ${BUILD_TAG}", tokenCredentialId: 'slack-token')
//            mail(subject: 'Staging Build', body: 'New Deployment to Staging', to: 'jenkins-mailing-list@mail.com')
          }
        }

        stage('QA') {
          when {
           branch 'qa'
          }
          steps {
            withAWS(region:'us-east-1',credentials:'s3-conquista') {
              s3Delete(bucket: 'conquista-frontend-qa', path:'**/*')
              s3Upload(bucket: 'conquista-frontend-qa', workingDir:'build', includePathPattern:'**/*', acl:'PublicRead');
            }
            //  slackSend (color: 'good', message: "[ SUCESSO ] Deploy aplicado com sucesso! \n\n ${BUILD_TAG}", tokenCredentialId: 'slack-token')
//            mail(subject: 'Staging Build', body: 'New Deployment to Staging', to: 'jenkins-mailing-list@mail.com')
          }
        }

        stage('Production') {
          when {
            branch 'master'
          }
          steps {
            withAWS(region:'us-east-1',credentials:'s3-conquista') {
              s3Delete(bucket: 'conquista-frontend-prod', path:'**/*')
              s3Upload(bucket: 'conquista-frontend-prod', workingDir:'build', includePathPattern:'**/*', acl:'PublicRead');
            }
            //  slackSend (color: 'good', message: "[ SUCESSO ] Deploy aplicado com sucesso! \n\n ${BUILD_TAG}", tokenCredentialId: 'slack-token')
//            mail(subject: 'Staging Build', body: 'New Deployment to Staging', to: 'jenkins-mailing-list@mail.com')
            }
          }
        }
      }
    }
}