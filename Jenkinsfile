pipeline {
    agent any
    
    environment {
        AWS_REGION = 'eu-west-1'  // region
        ECR_REGISTRY = '412128479181.dkr.ecr.eu-west-1.amazonaws.com'  //  ECR registry
        ECR_REPOSITORY = 'nginx-docker-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        AWS_CREDENTIAL_ID = 'aws-ecr-credentials'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out code from GitHub...'
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    dockerImage = docker.build("${ECR_REPOSITORY}:${IMAGE_TAG}")
                }
            }
        }
        
        stage('Authenticate with AWS ECR') {
            steps {
                echo 'Authenticating with AWS ECR...'
                script {
                    withAWS(credentials: "${AWS_CREDENTIAL_ID}", region: "${AWS_REGION}") {
                        sh """
                            aws ecr get-login-password --region ${AWS_REGION} | \
                            docker login --username AWS --password-stdin ${ECR_REGISTRY}
                        """
                    }
                }
            }
        }
        
        stage('Tag & Push Image to ECR') {
            steps {
                echo 'Tagging and pushing image to ECR...'
                script {
                    sh """
                        docker tag ${ECR_REPOSITORY}:${IMAGE_TAG} ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                        docker tag ${ECR_REPOSITORY}:${IMAGE_TAG} ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest
                        docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                        docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest
                    """
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                echo 'Cleaning up local images...'
                script {
                    sh """
                        docker rmi ${ECR_REPOSITORY}:${IMAGE_TAG} || true
                        docker rmi ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG} || true
                        docker rmi ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest || true
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
            echo "Image pushed to ECR: ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}
