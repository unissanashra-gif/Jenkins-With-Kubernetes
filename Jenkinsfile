pipeline {

    agent any

    environment {
        DOCKER_IMAGE = 'unisanashra/k8s-cicd-app'
        PATH = "/Users/afsarunisa/.docker/bin:${env.PATH}"
        DOCKER = 'docker'
        KUBECTL = 'kubectl'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "Running application tests..."

                    test -f app/index.html

                    grep "Hello from Kubernetes" app/index.html

                    echo "Tests passed successfully."
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "Building Docker image..."

                    ${DOCKER} build \
                        -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .

                    echo "Docker image built successfully."
                '''
            }
        }

        stage('Docker Push') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "Logging in to Docker Hub..."

                        echo "$DOCKER_PASSWORD" | ${DOCKER} login \
                            --username "$DOCKER_USERNAME" \
                            --password-stdin

                        echo "Pushing Docker image..."

                        ${DOCKER} push ${DOCKER_IMAGE}:${BUILD_NUMBER}

                        ${DOCKER} logout

                        echo "Docker image pushed successfully."
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Deploying application to Kind Kubernetes..."

                    sed "s|IMAGE_PLACEHOLDER|${DOCKER_IMAGE}:${BUILD_NUMBER}|g" \
                        kubernetes/deployment.yaml \
                        | ${KUBECTL} apply -f -

                    ${KUBECTL} apply -f kubernetes/service.yaml

                    echo "Kubernetes deployment applied successfully."
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Waiting for Kubernetes rollout..."

                    ${KUBECTL} rollout status \
                        deployment/k8s-cicd-app \
                        --timeout=120s

                    echo "Deployment:"
                    ${KUBECTL} get deployment k8s-cicd-app

                    echo "Pods:"
                    ${KUBECTL} get pods

                    echo "Service:"
                    ${KUBECTL} get service k8s-cicd-service

                    echo "Kubernetes deployment verified successfully."
                '''
            }
        }
    }

    post {

        success {
            echo 'CI/CD PIPELINE COMPLETED SUCCESSFULLY!'
        }

        failure {
            echo 'CI/CD PIPELINE FAILED.'
        }
    }
}
