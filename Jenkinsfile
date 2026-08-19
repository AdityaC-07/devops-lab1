pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = 'devops-lab1-app'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Python dependencies...'
                bat 'python -m pip install flake8 pytest'
            }
        }

        stage('Lint') {
            steps {
                echo 'Running flake8...'
                bat 'python -m flake8 app.py'
            }
        }

        stage('Unit Test') {
            steps {
                echo 'Running unit tests...'
                bat 'python -m pytest test_app.py -v'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                bat 'docker build -t devops-lab1-app:%BUILD_NUMBER% .'
            }
        }

        stage('ECR Login') {
            steps {
                echo 'Logging in to Amazon ECR...'
                bat '''
                    for /f "delims=" %%i in ('aws sts get-caller-identity --query Account --output text') do set AWS_ACCOUNT_ID=%%i
                    aws ecr get-login-password --region %AWS_REGION% | docker login --username AWS --password-stdin %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                echo 'Pushing Docker image to ECR...'
                bat '''
                    for /f "delims=" %%i in ('aws sts get-caller-identity --query Account --output text') do set AWS_ACCOUNT_ID=%%i

                    docker tag devops-lab1-app:%BUILD_NUMBER% %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%ECR_REPO%:%BUILD_NUMBER%

                    docker push %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%ECR_REPO%:%BUILD_NUMBER%

                    docker tag devops-lab1-app:%BUILD_NUMBER% %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%ECR_REPO%:latest

                    docker push %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%ECR_REPO%:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the stage logs.'
        }
    }
}
