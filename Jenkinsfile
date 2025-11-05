pipeline{
  agent any
  stages{
    stage ('zip_the_application'){
     steps{
        sh """ echo "in the zipping process"
            zip Devops
            """
        }
        }
     stage('create'){
        steps{
          sh '''
                aws s3api create-bucket --bucket my-demo-bucket --region ap-south-1 --create-bucket-configuration LocationConstraint=ap-south-1
                aws s3 cp Devops.zip s3://my-demo-bucket/
        '''
        }
     }
  }
}
