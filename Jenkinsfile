pipeline {
    agent any

    tools {
        git 'Default'  // Name your Git installation in Global Tool Config
    }

    environment {
        AWS_REGION     = 'us-east-1'
        AMI_ID         = 'ami-0ec18f6103c5e0491'
        INSTANCE_TYPE  = 't2.micro'
        KEY_NAME       = 'newjenkinskey'
        SECURITY_GROUP = 'sg-0ece4b3e66a57dd4d'
        SUBNET_ID      = 'subnet-0d08241fda0e0aa1f'
    }

    stages {
        stage('Launch EC2 Instance') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'jenkins-aws-start-stop',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    script {
                        env.INSTANCE_ID = sh(
                            script: '''
aws ec2 run-instances \
  --image-id $AMI_ID \
  --count 1 \
  --instance-type $INSTANCE_TYPE \
  --key-name $KEY_NAME \
  --security-group-ids $SECURITY_GROUP \
  --subnet-id $SUBNET_ID \
  --region $AWS_REGION \
  --query 'Instances[0].InstanceId' \
  --output text
''', returnStdout: true
                        ).trim()
                        echo "Launched EC2 instance: ${env.INSTANCE_ID}"
                    }
                }
            }
        }

        stage('Wait for Instance & Get IP') {
            steps {
                script {
                    sh "aws ec2 wait instance-running --instance-ids ${env.INSTANCE_ID} --region ${AWS_REGION}"
                    env.PUBLIC_IP = sh(
                        script: '''
aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --region $AWS_REGION \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text
''', returnStdout: true
                    ).trim()
                    echo "Public IP of instance: ${env.PUBLIC_IP}"
                }
            }
        }

        stage('Install Tomcat') {
            steps {
                sshagent(['jenkins-ec2-ssh-key']) {
                    sh '''
ssh -o StrictHostKeyChecking=no ec2-user@$PUBLIC_IP << 'EOF'
  sudo yum update -y
  sudo yum install -y java-1.8.0-openjdk wget
  wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.82/bin/apache-tomcat-9.0.82.tar.gz
  tar xzvf apache-tomcat-9.0.82.tar.gz
  sudo mv apache-tomcat-9.0.82 /opt/tomcat
  sudo /opt/tomcat/bin/startup.sh
EOF
'''
                }
                echo "Tomcat installed and started on EC2!"
            }
        }
    }

    post {
        always {
            echo "Pipeline completed. Remember to terminate ${env.INSTANCE_ID} when done."
        }
    }
}