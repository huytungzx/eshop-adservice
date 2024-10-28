/*
<< 변수 >> 치환 필요

<< ECR URI >>      => ex) 123456789012.dkr.ecr.us-east-1.amazonaws.com
ex) 123456789012.dkr.ecr.us-east-1.amazonaws.com/eshop-backend:latest
*/
pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: gradle
spec:
  containers:
  - name: gradle
    image: gradle:6.3.0-jdk11
    command:
    - cat
    tty: true
    env:
      # Define the environment variable
      - name: CRED
        valueFrom:
          configMapKeyRef:
            name: jenkinscred
            key: ECR_CREDENTIAL_JSON
  restartPolicy: Never
"""      
    }
  }
  environment {
    IMAGE_REGISTRY = "<< ECR URI >>"
  }
  stages {

    stage('Approval') {
      when {
        branch 'main'
      }
      steps {
        script {
          def plan = 'adservice CI'
          input message: "Do you want to build and push?",
              parameters: [text(name: 'Plan', description: 'Please review the work', defaultValue: plan)]
        }
      } 
    }

    stage('build and push docker image') {
      when {
        branch 'main'
      }             
      steps {
        container('gradle') {
          sh 'gradle jib --no-daemon --image ${IMAGE_REGISTRY}/eshop-adservice:latest -Djib.to.auth.username=AWS -Djib.to.auth.password=$CRED'  
        }
      }
      post {
        success { 
          slackSend(channel: '<< CHANNEL ID >>', color: 'good', message: 'adservice CI success')
        }
        failure {
          slackSend(channel: '<< CHANNEL ID >>', color: 'danger', message: 'adservice CI fail')
        }
      }
    }
  }
}