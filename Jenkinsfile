
pipeline {

    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '242627333029'
        ECR_REPOSITORY = 'demo-project'
        IMAGE_TAG = "${BUILD_NUMBER}"
        APP_INSTANCE_ID = 'i-0c800130b9e44357c'
    }

    stages {

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t demo-project:$IMAGE_TAG .'
            }
        }

        stage('ECR Authentication') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'aws-ecr-creds',
                usernameVariable: 'AWS_ACCESS_KEY_ID',
                passwordVariable: 'AWS_SECRET_ACCESS_KEY'
            )
        ]) {
            sh '''
                export AWS_DEFAULT_REGION=ap-south-1

                aws ecr get-login-password \
                --region ap-south-1 | docker login \
                --username AWS \
                --password-stdin \
                242627333029.dkr.ecr.ap-south-1.amazonaws.com
            '''
        }
    }
}
        stage('Tag Docker Image') {
            steps {
                sh '''
                    docker tag demo-project:$IMAGE_TAG \
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

       
stage('Deploy to Application EC2') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'aws-ecr-creds',
                usernameVariable: 'AWS_ACCESS_KEY_ID',
                passwordVariable: 'AWS_SECRET_ACCESS_KEY'
            )
        ]) {
            sh '''
                set -e

                export AWS_DEFAULT_REGION=$AWS_REGION

                ECR_URI="$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPOSITORY"

                echo "Deploying image: $ECR_URI:$IMAGE_TAG"

                COMMAND_ID=$(aws ssm send-command \
                    --region "$AWS_REGION" \
                    --instance-ids "$APP_INSTANCE_ID" \
                    --document-name "AWS-RunShellScript" \
                    --parameters "commands=[
                      \\"aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com\\",
                      \\"docker pull $ECR_URI:$IMAGE_TAG\\",
                      \\"docker stop demo-project || true\\",
                      \\"docker rm demo-project || true\\",
                      \\"docker run -d --restart unless-stopped --name demo-project -p 3000:3000 $ECR_URI:$IMAGE_TAG\\"
                    ]" \
                    --query "Command.CommandId" \
                    --output text)

                echo "SSM Command ID: $COMMAND_ID"

                for i in $(seq 1 60); do

                    STATUS=$(aws ssm get-command-invocation \
                        --region "$AWS_REGION" \
                        --command-id "$COMMAND_ID" \
                        --instance-id "$APP_INSTANCE_ID" \
                        --query "Status" \
                        --output text 2>/dev/null || true)

                    echo "Deployment status: $STATUS"

                    case "$STATUS" in
                        Success)
                            echo "Application deployed successfully!"
                            exit 0
                            ;;
                        Failed|Cancelled|TimedOut|Cancelling)
                            echo "Deployment failed!"
                            aws ssm get-command-invocation \
                                --region "$AWS_REGION" \
                                --command-id "$COMMAND_ID" \
                                --instance-id "$APP_INSTANCE_ID"
                            exit 1
                            ;;
                    esac

                    sleep 5
                done

                echo "Deployment timed out. Check SSM command status."
                exit 1
            '''
        }
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

