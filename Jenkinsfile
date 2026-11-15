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
                sh 'npm ci'
            }
        }

        stage('Run Tests') {
            steps {
                echo 'No automated tests configured yet. Skipping tests.'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t demo-pj:latest .'
            }
        }
    }
}
