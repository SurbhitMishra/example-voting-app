pipeline {
    agent { label 'worker' }

    environment {
        ECR_REGISTRY = '241919703477.dkr.ecr.ap-south-1.amazonaws.com/task3' // ECR registry URI
        ECR_REPO = 'task3' // ECR repository name
        DOCKER_TAG = "vote-v${BUILD_NUMBER}" // Unique tag using Jenkins BUILD_NUMBER
    }

    stages {
        stage('Authenticate with ECR') {
            steps {
                script {
                    echo 'Authenticating with ECR...'
                    sh """
                    aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    sh """
                    cd vote
                    docker build -t ${ECR_REGISTRY}/${ECR_REPO}:${DOCKER_TAG} .
                    """
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    echo 'Pushing Docker image to ECR...'
                    sh """
                    docker push ${ECR_REGISTRY}/${ECR_REPO}:${DOCKER_TAG}
                    """
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    echo 'Deploying application...'
                    sh """
                    # Stop and force remove the existing container (if any)
                    docker rm -f ${ECR_REPO} || true
                    
                    # Pull the latest image from ECR
                    docker pull ${ECR_REGISTRY}/${ECR_REPO}:${DOCKER_TAG}

                    # Run the new container (ensure the application listens on port 80)
                    docker run -d --name ${ECR_REPO} -p 80:80 ${ECR_REGISTRY}/${ECR_REPO}:${DOCKER_TAG}
                    """
                }
            }
        }
    }
}
