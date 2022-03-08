# jenkins-nodejs-first

jenkins 的 nodejs 寫法

現有範例

```
pipeline {
  agent {
    label 'master'
  }
  stages {
    stage('GitSCM') {
      environment {
        CODE_COMMIT_ID = "jenkins_api-at-807372858626"
        CODE_COMMIT_URL = "https://git-codecommit.ap-southeast-1.amazonaws.com/v1/repos/cms-fend-client"
      }
      steps {
        git branch: 'development', credentialsId: CODE_COMMIT_ID, url: CODE_COMMIT_URL
      }
    }
    stage('NodeJS Install') {
      steps{
        nodejs(nodeJSInstallationName: 'NodeJS14' ){
          sh "node -v"
          sh 'npm install'
        }
      }
      post {
        failure {
          SendAlarm("NodeJS Install Failed!!", "")
        }
      }
    }
    stage('NodeJS Build') {
      steps{
        nodejs(nodeJSInstallationName: 'NodeJS14' ){
          sh 'npm run build-development'
        }
      }
      post {
        failure {
          SendAlarm("NodeJS Build Failed!!", "")
        }
      }
    }
    stage('Deploy') {
      environment {
        ACCESS = "'ssh -o StrictHostKeyChecking=no -i ~/.ssh/qa.key'"
        GIT_TAG_COMMIT = sh(script: 'git log -1 --pretty=%B', returnStdout: true).trim()
      }
      steps{
        sh "rsync --include '.*' -avpz --chmod=775 -e ${ACCESS} ./dist/ ubuntu@172.31.27.16:/var/www/html/cms-fend-client/"
      }
      post {
        failure {
          SendAlarm("上傳失敗!!", GIT_TAG_COMMIT)
        }
        success {
          slackSend (color:'#0000FF', message:"CMS控端後台-前端 佈署成功!!", channel: 'cms-jenkins通知警報區', tokenCredentialId: 'AAAAAAA')
        }
      }
    }
  }
}

def SendAlarm( caused = "", git_msg = "" ){
  slackSend (color:'#FF0000', message:"[CMS封測站警報]!!\n\t主因:${caused}\n\t\${git_msg}", channel: 'cms-jenkins通知警報區', tokenCredentialId: 'AAAAAAA')
}

```
