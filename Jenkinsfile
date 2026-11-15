pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/gaurav00612/jenkins-docker-learning.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
            sudo apt-get update
            sudo apt-get install -y nodejs npm
        '''
                sh 'npm ci'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t demo-pj:latest .'
            }
        }
    }
}
