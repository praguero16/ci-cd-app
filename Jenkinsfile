pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "394811801190.dkr.ecr.ap-south-1.amazonaws.com/ci-cd-app"
        IMAGE_TAG = "latest"
        EC2_IP = "13.234.78.17"
    }

    stages {

        stage("Checkout") {
            steps {
                checkout scm
            }
        }

        stage("Install Dependencies") {
            steps {
                echo "Skipping npm install ? handled inside Dockerfile"
            }
        }

        stage("Build Docker Image") {
            steps {
                script {
                    dockerImage = docker.build("${ECR_REPO}:${IMAGE_TAG}")
                }
            }
        }

        stage("Login to ECR") {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    bat """
                        aws ecr get-login-password --region %AWS_REGION% ^
                        | docker login --username AWS --password-stdin %ECR_REPO%
                    """
                }
            }
        }

        stage("Push to ECR") {
            steps {
                script {
                    dockerImage.push()
                }
            }
        }

        stage("Deploy to EC2") {
    		steps {
        		withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'SSH_KEY')]) {

        			bat """
                			rem --- FIX WINDOWS SSH KEY PERMISSIONS ---
                			icacls %SSH_KEY% /inheritance:r
                			icacls %SSH_KEY% /grant:r "Administrators:F"

					rem --- LOGIN TO ECR ON EC2 ---
                			ssh -o StrictHostKeyChecking=no -i %SSH_KEY% ubuntu@%EC2_IP% "aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 394811801190.dkr.ecr.ap-south-1.amazonaws.com"


			                rem --- DEPLOY TO EC2 ---
                			ssh -o StrictHostKeyChecking=no -i %SSH_KEY% ubuntu@${EC2_IP} "docker pull ${ECR_REPO}:${IMAGE_TAG}"
                			ssh -o StrictHostKeyChecking=no -i %SSH_KEY% ubuntu@${EC2_IP} "docker stop ci-cd-app || true"
                			ssh -o StrictHostKeyChecking=no -i %SSH_KEY% ubuntu@${EC2_IP} "docker rm ci-cd-app || true"
                			ssh -o StrictHostKeyChecking=no -i %SSH_KEY% ubuntu@${EC2_IP} "docker run -d -p 80:3000 --name ci-cd-app ${ECR_REPO}:${IMAGE_TAG}"
            			"""
        		}
    		}
	}


    }
}
