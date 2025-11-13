pipeline {
    agent any

    environment {
        IMAGE_NAME = "benjaminssempala/petclinic2"
        TAG = "latest"
    }

    stages {
        stage('Setup Environment') {
            steps {
                echo "🔧 Setting up environment..."

                sh '''
                echo "--- Checking prerequisites ---"
                whoami
                echo "--- PATH ---"
                echo $PATH

                # Ensure containerd socket exists
                if [ ! -S /run/containerd/containerd.sock ]; then
                  echo "⚠️ containerd socket not found, trying to restart containerd"
                   systemctl restart containerd || true
                  sleep 5
                fi

                # Install nerdctl if missing
                if ! command -v nerdctl >/dev/null 2>&1; then
                  echo "📦 Installing nerdctl..."
                  ARCH=$(uname -m)
                  VERSION="1.7.6"
                  curl -L "https://github.com/containerd/nerdctl/releases/download/v${VERSION}/nerdctl-${VERSION}-linux-${ARCH}.tar.gz" -o nerdctl.tar.gz
                  tar -zxvf nerdctl.tar.gz -C /usr/local/bin
                  rm -f nerdctl.tar.gz
                else
                  echo "✅ nerdctl already installed"
                fi

                # Confirm container runtime
                nerdctl --version || echo "nerdctl not found even after install"
                '''
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/BenjaminSsempala/spring-petclinic.git'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    echo "🏗️ Building container image..."
                    sh '''
                    if command -v nerdctl >/dev/null 2>&1; then
                      nerdctl build -t $IMAGE_NAME:$TAG .
                    else
                      echo "❌ nerdctl not available, aborting build"
                      exit 1
                    fi
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                echo "🧪 Running tests..."
                sh 'echo "Pretend tests run here..."'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DOCKER_HUB_CREDENTIALS', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                    echo "🚀 Logging in to Docker Hub..."
                    nerdctl login -u "$USERNAME" -p "$PASSWORD"
                    nerdctl push $IMAGE_NAME:$TAG
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Build finished.'
        }
        failure {
            echo '❌ Build failed. Check logs above.'
        }
    }
}
