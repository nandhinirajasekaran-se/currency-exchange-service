pipeline {
  agent any

  environment {
    AWS_REGION = 'us-west-2'
    ECR_REPO = '540607980171.dkr.ecr.us-west-2.amazonaws.com/currency-exchange-service'
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build JAR with Maven') {
      steps {
        sh 'mvn clean package -DskipTests'
      }
    }

    stage('Verify JAR exists') {
      steps {
        sh 'ls -lh target/'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $ECR_REPO:$IMAGE_TAG .'
      }
    }

    stage('Push to ECR') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-jenkins']]) {
          sh '''
            aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
            docker push $ECR_REPO:$IMAGE_TAG
          '''
        }
      }
    }
    stage(){
      steps {
          argocd login a3cf9f226fb2344eb9c97b23a3ad46cf-2040838917.us-west-2.elb.amazonaws.com --insecure --sso-token $ARGOCD_AUTH_TOKEN
          argocd app sync currency-exchange
          argocd app wait currency-exchange --health --timeout 300
      }
    }
  }
}
