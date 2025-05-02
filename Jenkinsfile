pipeline {
    agent any

    environment {
        IMAGE_NAME = 'subhash45/outage-communication-app'
        DOCKER_CREDENTIALS = credentials('docker-hub-creds')
        K8S_DEPLOY_FILE = 'k8s/deployment.yaml'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/awsSubhash/argocdgithub.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    IMAGE_TAG = "${env.BUILD_NUMBER}"
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS) {
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Update YAML') {
            steps {
                script {
                    sh """
                    sed -i 's|image: .*$|image: ${IMAGE_NAME}:${IMAGE_TAG}|' ${K8S_DEPLOY_FILE}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f ${K8S_DEPLOY_FILE}"
            }
        }
    }
}

