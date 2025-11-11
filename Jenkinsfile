pipeline {
  agent any
  environment {
    GIT_BRANCH  = 'main'
    REPO_URL    = 'https://github.com/Random139/DevopsTest'
    S3_BUCKET   = 'demo-bucket-139-jenkins'
    S3_KEY      = 'emp_det.zip'
    LAMBDA_NAME = 'Emp_details'
    AWS_REGION  = 'ap-south-1'
    ROLE_ARN    = 'arn:aws:iam::873046390774:role/Lambda_db_access'
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: "${GIT_BRANCH}", url: "${REPO_URL}"
      }
    }
    stage('check'){
      steps('checking'){
        sh 'aws sts get-caller-identity --region ap-south-1'
      }
    }
    stage('Package') {
      steps {
        sh 'zip -r emp_det.zip app.py'
        archiveArtifacts artifacts: 'emp_det.zip'
      }
    }
    stage('Upload to S3') {
      steps {
        withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding',
          credentialsId: 'aws-deploy-creds',
          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
          sh "aws s3 cp emp_det.zip s3://${S3_BUCKET}/${S3_KEY} --region ${AWS_REGION}"
        }
      }
    }
    stage('Deploy Lambda') {
      steps {
        script {
          withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'aws-deploy-creds',
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
          ]]) {
            def ret = sh(
              script: "aws lambda update-function-code --function-name ${LAMBDA_NAME} --s3-bucket ${S3_BUCKET} --s3-key ${S3_KEY} --region ${AWS_REGION}",
              returnStatus: true
            )
            if (ret != 0) {
              sh "aws lambda create-function --function-name ${LAMBDA_NAME} --runtime python3.9 --role ${ROLE_ARN} --handler lambda_function.handler --code S3Bucket=${S3_BUCKET},S3Key=${S3_KEY} --region ${AWS_REGION}"
            }
          }
        }
      }
    }
  }
  post {
    success { echo "Deployment succeeded" }
    failure { echo "Deployment failed" }
  }
}
