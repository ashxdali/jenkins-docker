pipeline {
    agent any

    environment {
        APP_NAME = 'jenkins-docker-demo'
        IMAGE_NAME = 'ashxdali/jenkins-docker-demo'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                    python3 -m py_compile app.py
                    echo "Application build completed successfully."
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    pip3 install flask pytest --break-system-packages
                    pytest
                '''
            }
        }

        stage('Package') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}
