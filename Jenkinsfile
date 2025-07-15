pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AMI_ID = 'ami-0ec18f6103c5e0491'  // Replace with your preferred Amazon Linux AMI
        INSTANCE_TYPE = 't2.micro'
        KEY_NAME = 'newjenkinskey'
        SECURITY_GROUP = 'sg-0ece4b3e66a57dd4d'
        SUBNET_ID = 'subnet-0d08241fda0e0aa1f'
    }

    stages {
        stage('Launch EC2 Instance') {
            steps {
                script {
                    def result = sh(script: """
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
                    """, returnStdout: true).trim()

                    env.INSTANCE_ID = result
                    echo "Launched EC2 Instance: ${env.INSTANCE_ID}"
                }
            }
        }

        stage('Wait for Instance & Get IP') {
            steps {
                script {
                    sh "aws ec2 wait instance-running --instance-ids ${env.INSTANCE_ID} --region $AWS_REGION"

                    def public_ip = sh(script: """
                        aws ec2 describe-instances \
                            --instance-ids ${env.INSTANCE_ID} \
                            --query 'Reservations[0].Instances[0].PublicIpAddress' \
                            --output text \
                            --region $AWS_REGION
                    """, returnStdout: true).trim()

                    env.PUBLIC_IP = public_ip
                    echo "Instance Public IP: ${env.PUBLIC_IP}"
                }
            }
        }

        stage('Install Tomcat') {
            steps {
                script {
                    sh """
                        ssh -o StrictHostKeyChecking=no -i /path/to/your/private-key.pem ec2-user@${env.PUBLIC_IP} <<EOF
                        sudo yum update -y
                        sudo yum install -y java-1.8.0-openjdk
                        wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.82/bin/apache-tomcat-9.0.82.tar.gz
                        tar xzvf apache-tomcat-9.0.82.tar.gz
                        sudo mv apache-tomcat-9.0.82 /opt/tomcat
                        sudo sh /opt/tomcat/bin/startup.sh
                        EOF
                    """
                    echo "Tomcat installed and started on EC2!"
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed."
        }
    }
}