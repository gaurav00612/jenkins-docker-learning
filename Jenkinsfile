
pipeline {

    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '242627333029'
        ECR_REPOSITORY = 'demo-project'
        IMAGE_TAG = 'v1'
    }

    stages {

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t student-app:v1 .'
            }
        }

        stage('ECR Authentication') {
            steps {
                sh '''
                    aws ecr get-login-password \
                    --region $AWS_REGION | docker login \
                    --username AWS \
                    --password-stdin \
                    $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                    docker tag student-app:v1 \
                    $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY:$IMAGE_TAG
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    docker push \
                    $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        success {
            echo 'Docker image successfully pushed to Amazon ECR!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}

