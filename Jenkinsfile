pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub')
        IMAGE_NAME = "benjaminssempala/petclinic2:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/BenjaminSsempala/spring-petclinic.git'
            }
        }

        stage('Check Container Runtime') {
            steps {
                sh '''
                echo "WHOAMI: $(whoami)"
                echo "--- PATH ---"
                echo $PATH
                echo "--- Checking for Docker or nerdctl ---"
                which docker && docker --version || echo "Docker not found"
                which nerdctl && nerdctl --version || echo "nerdctl not found"
                echo "--- Checking containerd socket ---"
                if [ -S /run/containerd/containerd.sock ]; then
                    echo "Containerd socket available."
                else
                    echo "❌ containerd socket missing!"
                fi
                '''
            }
        }

        stage('Build Image') {
            steps {
                script {
                    sh """
                    echo "Building Docker image using nerdctl..."
                    sudo nerdctl build -t ${IMAGE_NAME} .
                    """
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    echo "Running Maven tests..."
                    sh 'mvn test'
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    sh """
                    echo "Logging in to Docker Hub..."
                    echo "${DOCKER_HUB_CREDENTIALS_PSW}" | sudo nerdctl login -u "${DOCKER_HUB_CREDENTIALS_USR}" --password-stdin
                    echo "Pushing image to Docker Hub..."
                    sudo nerdctl push ${IMAGE_NAME}
                    """
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh "sudo nerdctl rmi ${IMAGE_NAME} || true"
            }
        }
    }

    post {
        success {
            echo "Build, Test, and Push successful!"
        }
        failure {
            echo "Build failed. Check logs above."
        }
    }
}
