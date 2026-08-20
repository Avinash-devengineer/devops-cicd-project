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

        stage('Get Version') {
            steps {
                script {
                    env.GIT_SHORT_COMMIT = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()

                    env.IMAGE_TAG = "build-${BUILD_NUMBER}-git-${GIT_SHORT_COMMIT}"

                    echo "Git Commit: ${GIT_SHORT_COMMIT}"
                    echo "Docker Image Tag: ${IMAGE_TAG}"
                }
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
                echo "Building Docker image: ${IMAGE_TAG}"

                sh '''
                    docker build \
                        -t ${DOCKER_IMAGE}:${IMAGE_TAG} \
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

                        docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying ${DOCKER_IMAGE}:${IMAGE_TAG}"

                sh '''
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${APP_PORT}:${CONTAINER_PORT} \
                        ${DOCKER_IMAGE}:${IMAGE_TAG}

                    echo "Waiting for application to start..."
                    sleep 5
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking application health...'

                sh '''
                    curl --fail --silent --show-error \
                        http://localhost:${APP_PORT} > /tmp/healthcheck.html

                    grep -q "DevOps CI/CD Dashboard" /tmp/healthcheck.html

                    echo "Health check passed"
                    echo "Application is running successfully"
                '''
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'CI/CD PIPELINE SUCCESSFUL'
            echo '========================================'
            echo "Git Commit: ${GIT_SHORT_COMMIT}"
            echo "Docker Image: ${DOCKER_IMAGE}:${IMAGE_TAG}"
            echo "Docker Latest: ${DOCKER_IMAGE}:latest"
            echo "Application: http://localhost:${APP_PORT}"
            echo '========================================'
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
