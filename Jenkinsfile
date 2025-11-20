pipeline {
    agent any
    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "394811801190.dkr.ecr.ap-south-1.amazonaws.com/ci-cd-app"
        IMAGE_TAG = "latest"
        EC2_IP = "13.234.78.17"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${ECR_REPO}:${IMAGE_TAG}")
                }
            }
        }
        stage('Login to ECR') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION \
                        | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }
        stage('Push to ECR') {
            steps {
                script {
                    dockerImage.push()
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ['ec2-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@$EC2_IP "
                            docker pull ${ECR_REPO}:${IMAGE_TAG} &&
                            docker stop ci-cd-app || true &&
                            docker rm ci-cd-app || true &&
                            docker run -d -p 80:3000 --name ci-cd-app ${ECR_REPO}:${IMAGE_TAG}
                        "
                    '''
                }
            }
        }
    }
}
