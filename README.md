# jenkins-nodejs-first <!-- omit in toc -->

jenkins 的 nodejs 寫法，列出現有範例

- [Linux](#linux)
- [Node & Windows (Bash)](#node--windows-bash)
- [prod-rg-crawler-python](#prod-rg-crawler-python)

## Linux

development

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

release

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
        git branch: 'release', credentialsId: CODE_COMMIT_ID, url: CODE_COMMIT_URL
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
          sh 'npm run build-release'
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
        ACCESS = "'ssh -o StrictHostKeyChecking=no -i ~/.ssh/prod_jumper'"
        GIT_TAG_COMMIT = sh(script: 'git log -1 --pretty=%B', returnStdout: true).trim()
      }
      steps{
        sh "rsync --include '.*' -avpz --chmod=775 -e ${ACCESS} ./dist/ ubuntu@13.251.158.118:/var/www/html/cms-fend-client/"
      }
      post {
        failure {
          SendAlarm("上傳失敗!!", GIT_TAG_COMMIT)
        }
        success {
          slackSend (color:'#0000FF', message:"[正式站測試線]CMS控端後台-前端 佈署成功!!", channel: 'cms-jenkins通知警報區', tokenCredentialId: 'AAAAAAA')
        }
      }
    }
  }
}

def SendAlarm( caused = "", git_msg = "" ){
  slackSend (color:'#FF0000', message:"[CMS正式站警報]!!\n\t主因:${caused}\n\t\${git_msg}", channel: 'cms-jenkins通知警報區', tokenCredentialId: 'AAAAAAA')
}

```

master

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
        git branch: 'master', credentialsId: CODE_COMMIT_ID, url: CODE_COMMIT_URL
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
          sh 'npm run build'
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
        ACCESS = "'ssh -o StrictHostKeyChecking=no -i ~/.ssh/prod_jumper'"
        GIT_TAG_COMMIT = sh(script: 'git log -1 --pretty=%B', returnStdout: true).trim()
      }
      steps{
        sh "rsync --include '.*' -avpz --chmod=775 -e ${ACCESS} ./dist/ ubuntu@13.251.158.117:/var/www/html/cms-fend-client/"
      }
      post {
        failure {
          SendAlarm("上傳失敗!!", GIT_TAG_COMMIT)
        }
        success {
          slackSend (color:'#0000FF', message:"[正式站]CMS控端後台-前端 佈署成功!!", channel: 'cms-jenkins通知警報區', tokenCredentialId: 'AAAAAAA')
        }
      }
    }
  }
}

def SendAlarm( caused = "", git_msg = "" ){
  slackSend (color:'#FF0000', message:"[CMS正式站警報]!!\n\t主因:${caused}\n\t\${git_msg}", channel: 'cms-jenkins通知警報區', tokenCredentialId: 'AAAAAAA')
}

```

## Node & Windows (Bash)

```
node('qa-office-auto-tester') {
    stage('GitSCM') {
        bat """
            cd cms-autotest-katalon
            git pull
        """
    }
    stage('NodeJS') {
        bat """
            cd cms-autotest-katalon
            node -v
            pnpm install
            npx cypress info
        """
    }
    stage('充值1元') {
        bat """
            cd cms-autotest-katalon
            pnpm transfer-all-1
        """
    }
}

def SendAlarm( caused = "", git_msg = "" ){
  slackSend (color:'#FF0000', message:"[CMS封測站警報]!!\n\t主因:${caused}\n\t\${git_msg}", channel: 'cms-jenkins通知警報區', tokenCredentialId: 'AAAAAAA')
}
```

## prod-rg-crawler-python

```
import groovy.transform.Field
import groovy.lang.Binding

@Field def SSH = ""
node('master') {

  stage('拉取專案') {
    // SSH & rsync access op tions
    relay_account = "ec2-user@13.251.158.107"
    SSH = "ssh -o StrictHostKeyChecking=no -i ~/.ssh/prod_jumper  $relay_account"

    //params.branch
    git credentialsId: "jenkins_api-at-807372858626", branch: params.branch, url:"https://git-codecommit.ap-southeast-1.amazonaws.com/v1/repos/provider-crawler"
  }

  stage('準備跳板') {
    sh("rsync -avpz -e 'ssh -o StrictHostKeyChecking=no -i ~/.ssh/prod_jumper' ./ ${relay_account}:/tmp/jenkins/provider-crawler/")
  }

  stage('佈署到機器'){
    sh("$SSH \"rsync -avpz -e 'ssh -o StrictHostKeyChecking=no -i /home/ec2-user/.ssh/deploy_key' /tmp/jenkins/provider-crawler/* ec2-user@13.251.158.106:/home/ec2-user/provider-crawler/\"")
  }
}
```

stage-isc-python-crawler

```
import groovy.transform.Field
import groovy.lang.Binding

@Field def SSH = ""
node('master') {

  stage('拉取專案') {
    // SSH & rsync access op tions
    relay_account = "ec2-user@172.31.17.96"
    SSH = "ssh -o StrictHostKeyChecking=no -i ~/.ssh/bench_mark.key $relay_account"

    //params.branch
    git credentialsId: "jenkins_api-at-807372858626", branch: params.branch, url:"https://git-codecommit.ap-southeast-1.amazonaws.com/v1/repos/provider-crawler"
  }

  stage('準備跳板') {
    sh("rsync -avpz -e 'ssh -o StrictHostKeyChecking=no -i ~/.ssh/bench_mark.key' ./ ${relay_account}:/tmp/jenkins/provider-crawler/")
  }

  stage('佈署到機器'){
    sh("$SSH \"rsync -avpz -e 'ssh -o StrictHostKeyChecking=no -i /home/ec2-user/.ssh/master.private.rsa' /tmp/jenkins/provider-crawler/* ec2-user@172.31.21.198:/home/ec2-user/provider-crawler/\"")
  }

}
```
