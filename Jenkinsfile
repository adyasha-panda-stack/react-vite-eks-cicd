pipeline {
    agent any

    tools {
        nodejs 'Node22'
    }

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = 'react-vite-app'
        ACCOUNT_ID = '738247188327'
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build React App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${ECR_REPO}:${IMAGE_TAG} .'
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds'
                ]]) {

                    sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login \
                    --username AWS \
                    --password-stdin \
                    ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                docker tag ${ECR_REPO}:${IMAGE_TAG} ${IMAGE_URI}
                docker push ${IMAGE_URI}
                '''
            }
        }

        stage('Deploy to Amazon EKS') {
            steps {
                sh '''
                kubectl set image deployment/react-vite-app \
                react-vite=${IMAGE_URI} \
                -n react-app
                '''
            }
        }

        stage('Verify Rollout') {
            steps {
                sh '''
                kubectl rollout status deployment/react-vite-app \
                -n react-app
                '''
            }
        }
    }

    post {

        success {
            echo 'Deployment Successful'
        }

        failure {
            echo 'Deployment Failed'
        }
    }
}
