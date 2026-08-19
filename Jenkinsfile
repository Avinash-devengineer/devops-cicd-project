pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "avinashdevengineer/devops-cicd-app"
        CONTAINER_NAME = "devops-cicd-container"
        APP_PORT = "8082"
        CONTAINER_PORT = "80"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Test') {
            steps {
                echo 'Running application test...'
                sh '''
                    test -f app/index.html
                    grep -q "DevOps CI/CD Dashboard" app/index.html
                    echo "Application test passed"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh '''
                    docker build \
                    -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                    -t ${DOCKER_IMAGE}:latest \
                    .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'

                sh '''
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${APP_PORT}:${CONTAINER_PORT} \
                        ${DOCKER_IMAGE}:${BUILD_NUMBER}

                    sleep 5

                    curl -f http://localhost:${APP_PORT}

                    echo "Deployment successful"
                '''
            }
        }
    }

    post {
        success {
            echo '========================================'
            echo 'CI/CD PIPELINE SUCCESSFUL'
            echo '========================================'
            echo "Docker Image: ${DOCKER_IMAGE}:${BUILD_NUMBER}"
            echo "Application: http://localhost:${APP_PORT}"
        }

        failure {
            echo '========================================'
            echo 'CI/CD PIPELINE FAILED'
            echo '========================================'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
