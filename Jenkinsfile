
pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '588804759279'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

        USER_SERVICE_IMAGE = "${ECR_REGISTRY}/ecommerce-user-service"
        PRODUCT_SERVICE_IMAGE = "${ECR_REGISTRY}/ecommerce-product-service"
        ORDER_SERVICE_IMAGE = "${ECR_REGISTRY}/ecommerce-order-service"

        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub'
                checkout scm
            }
        }

        stage('Build User Service') {
            steps {
                dir('user-service') {
                    sh 'docker build -t ${USER_SERVICE_IMAGE}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Build Product Service') {
            steps {
                dir('product-service') {
                    sh 'docker build -t ${PRODUCT_SERVICE_IMAGE}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Build Order Service') {
            steps {
                dir('order-service') {
                    sh 'docker build -t ${ORDER_SERVICE_IMAGE}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Authenticate with Amazon ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region "$AWS_REGION" |
                    docker login \
                      --username AWS \
                      --password-stdin "$ECR_REGISTRY"
                '''
            }
        }

        stage('Push Images to Amazon ECR') {
            steps {
                sh '''
                    docker push "$USER_SERVICE_IMAGE:$IMAGE_TAG"
                    docker push "$PRODUCT_SERVICE_IMAGE:$IMAGE_TAG"
                    docker push "$ORDER_SERVICE_IMAGE:$IMAGE_TAG"
                '''
            }
        }
    }

    post {
        success {
            echo "CI pipeline completed successfully."
            echo "Docker images pushed with tag: ${IMAGE_TAG}"
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}

