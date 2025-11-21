pipeline {
    agent any

    environment {
        REGISTRY = "docker.io"
        IMAGE_NAME = "saiprasad361/frontend"
        KUBECONFIG = "/etc/rancher/rke2/rke2.yaml"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/DeMiGoD013/frontend-task.git'
            }
        }

        stage('Build Image') {
            steps {
                sh """
                sudo podman build -t ${IMAGE_NAME}:latest .
                """
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo "$DOCKER_PASS" | sudo podman login -u "$DOCKER_USER" --password-stdin docker.io
                    """
                }
            }
        }

        stage('Push Image') {
            steps {
                sh """
                sudo podman push ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                # Apply manifests
                kubectl --kubeconfig=${KUBECONFIG} apply -f k8s/deployment.yaml
                kubectl --kubeconfig=${KUBECONFIG} apply -f k8s/service.yaml

                # Force update image to latest
                kubectl --kubeconfig=${KUBECONFIG} set image deployment/frontend frontend=${IMAGE_NAME}:latest --record
                """
            }
        }
    }
}
