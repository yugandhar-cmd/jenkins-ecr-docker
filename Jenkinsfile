pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '168287842191'
        AWS_REGION     = 'ap-south-1'
        ECR_REPO       = 'jenkins-ecr-push'

        IMAGE_NAME     = 'jenkins-ecr-push'
        IMAGE_TAG      = "${BUILD_NUMBER}"

        ECR_REGISTRY   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        ECR_URI        = "${ECR_REGISTRY}/${ECR_REPO}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .

                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
                        ${ECR_URI}:${IMAGE_TAG}

                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
                        ${ECR_URI}:latest
                '''
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: 'aws-ecr-credentials']
                ]) {
                    sh '''
                        aws ecr get-login-password \
                            --region ${AWS_REGION} | \
                        docker login \
                            --username AWS \
                            --password-stdin ${ECR_REGISTRY}
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker push ${ECR_URI}:${IMAGE_TAG}
                    docker push ${ECR_URI}:latest
                '''
            }
        }
    }

    post {
        success {
            echo "Docker image successfully pushed to AWS ECR!"
            echo "Image: ${ECR_URI}:${IMAGE_TAG}"
        }

        failure {
            echo "Docker image push failed."
        }
    }
}
