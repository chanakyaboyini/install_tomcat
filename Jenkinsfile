pipeline {
  agent any

  environment {
    AWS_REGION = 'us-east-1'
  }

  stages {
    stage('Launch EC2') {
      steps {
        // Binds AWS_ACCESS_KEY_ID & AWS_SECRET_ACCESS_KEY
        withCredentials([usernamePassword(
           credentialsId: 'jenkins-aws-start-stop',
           usernameVariable: 'AWS_ACCESS_KEY_ID',
           passwordVariable: 'AWS_SECRET_ACCESS_KEY'
        )]) {
          sh '''
            aws ec2 run-instances \
              --image-id ami-0ec18f6103c5e0491 \
              --count 1 \
              --instance-type t2.micro \
              --key-name newjenkinskey \
              --security-group-ids sg-0ece4b3e66a57dd4d \
              --subnet-id subnet-0d08241fda0e0aa1f \
              --region $AWS_REGION \
              --query Instances[0].InstanceId \
              --output text
          '''
        }
      }
    }

    // … your “Wait for Instance” and “Install Tomcat” stages …
  }

  post {
    always {
      echo "Pipeline execution completed."
    }
  }
}