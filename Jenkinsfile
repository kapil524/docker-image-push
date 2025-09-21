pipeline {
    agent any
    environment {
        DOCKERHUB = credentials('kmanwani-dockerhub')  // Docker Hub creds
        IMAGE_NAME = "flask-image"
        REPO = "kmanwani"
        CONTAINER_NAME = "flask-app"
        HOST_PORT = "80"
        CONTAINER_PORT = "8080"
        REGION = "ap-south-1"
        AMI_ID = "ami-02d26659fd82cf299"
        INSTANCE_TYPE = "t2.micro"
        VPC_ID = "vpc-0c87ced4dceaa7034"
        SUBNET_ID = "subnet-0d3310d1bf0a05c9f"
        KEY_NAME = "jenkins-ec2-key"  // EC2 Key Pair name
        SSH_KEY_PATH = "/var/lib/jenkins/.ssh/jenkins-ec2-key.pem"  // Private key on Jenkins
    }
    stages {
        stage('Build & Push Docker Image') {
            steps {
                sh 'docker build -t $REPO/$IMAGE_NAME:$BUILD_NUMBER .'
                sh 'echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin'
                sh 'docker push $REPO/$IMAGE_NAME:$BUILD_NUMBER'
            }
        }

        stage('Create EC2 & Deploy Container') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials-kapil'
                ]]) {
                    script {
                        // 1. Create Security Group
                        SG_ID = sh(script: """aws ec2 create-security-group \
                            --group-name flask-sg-$BUILD_NUMBER \
                            --description "Flask SG for Jenkins deployment" \
                            --vpc-id $VPC_ID \
                            --region $REGION \
                            --query 'GroupId' --output text""", returnStdout: true).trim()
                        echo "Created Security Group: ${SG_ID}"

                        // Add port 80 inbound
                        sh "aws ec2 authorize-security-group-ingress --group-id ${SG_ID} --protocol tcp --port 80 --cidr 0.0.0.0/0 --region $REGION"

                        // 2. Launch EC2 instance with Docker installation via user-data
                        USER_DATA = '''#!/bin/bash
                        apt-get update -y
                        apt-get install -y docker.io
                        systemctl start docker
                        systemctl enable docker
                        usermod -aG docker ubuntu
                        '''
                        INSTANCE_ID = sh(script: """aws ec2 run-instances \
                            --image-id $AMI_ID \
                            --count 1 \
                            --instance-type $INSTANCE_TYPE \
                            --key-name $KEY_NAME \
                            --security-group-ids ${SG_ID} \
                            --subnet-id $SUBNET_ID \
                            --region $REGION \
                            --user-data "$USER_DATA" \
                            --query 'Instances[0].InstanceId' --output text""", returnStdout: true).trim()
                        echo "EC2 Instance Created: ${INSTANCE_ID}"

                        // 3. Wait for instance to be ready
                        sh "aws ec2 wait instance_status_ok --instance-ids ${INSTANCE_ID} --region $REGION"

                        // 4. Get public IP
                        PUBLIC_IP = sh(script: """aws ec2 describe-instances \
                            --instance-ids ${INSTANCE_ID} \
                            --query 'Reservations[0].Instances[0].PublicIpAddress' \
                            --output text --region $REGION""", returnStdout: true).trim()
                        echo "EC2 Public IP: ${PUBLIC_IP}"

                        // 5. SSH and deploy container
                        sh """
                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY_PATH ubuntu@${PUBLIC_IP} \\
                            "docker login -u $DOCKERHUB_USR -p $DOCKERHUB_PSW && \\
                             docker stop $CONTAINER_NAME || true && \\
                             docker rm $CONTAINER_NAME || true && \\
                             docker pull $REPO/$IMAGE_NAME:$BUILD_NUMBER && \\
                             docker run -d --name $CONTAINER_NAME -p $HOST_PORT:$CONTAINER_PORT $REPO/$IMAGE_NAME:$BUILD_NUMBER"
                        """
                    }
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
