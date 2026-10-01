pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = 'ashxdali/jenkins-docker'
        IMAGE_TAG = "${BUILD_NUMBER}"
        TEST_CONTAINER = "jenkins-review-${BUILD_NUMBER}"
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
                    pip3 install -r requirements.txt --break-system-packages
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

        stage('Security Scan') {
            steps {
                sh '''
                    trivy image --severity HIGH,CRITICAL --exit-code 0 ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy & Verify') {
            steps {
                sh '''
                    docker rm -f ${TEST_CONTAINER} 2>/dev/null || true

                    docker run -d \
                        --name ${TEST_CONTAINER} \
                        --security-opt=no-new-privileges:true \
                        --cap-drop=ALL \
                        -p 5002:5000 \
                        ${IMAGE_NAME}:${IMAGE_TAG}

                    sleep 5

                    echo "Checking application health..."
                    curl --fail http://localhost:5002/health

                    echo "Checking container user..."
                    docker exec ${TEST_CONTAINER} whoami | grep -q appuser

                    echo "Deployment verification successful."
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker rm -f ${TEST_CONTAINER} 2>/dev/null || true'
        }

        success {
            echo 'Production readiness pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}























































