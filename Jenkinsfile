pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub')
        // SONARQUBE_ENV = credentials('sonarqube-token') 
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/BenjaminSsempala/spring-petclinic.git'
            }
        }

        // stage('Build Docker Image') {
        //     steps {
        //         script {
        //             sh '''
        //             docker build -t benjaminssempala/petclinic2:latest .
        //             '''
        //         }
        //     }
        // }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    echo "🐳 Checking Docker installation..."

                    if ! command -v docker >/dev/null 2>&1; then
                    echo "📦 Installing Docker..."
                    apt-get update -y
                    apt-get install -y ca-certificates curl gnupg lsb-release

                    # Add Docker’s official GPG key and repository
                    mkdir -p /etc/apt/keyrings
                    curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
                    echo \
                        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
                        $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list

                    sudo apt-get update -y
                    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

                    echo "✅ Docker installed successfully!"
                    else
                    echo "✅ Docker already installed."
                    docker --version
                    fi

                    echo "🏗️ Building Docker image..."
                    docker build -t benjaminssempala/petclinic2:latest .

                    echo "✅ Build complete!"
                    '''
                }
            }
        }

        

        stage('Test') {
            steps {
                script {
                    echo "Running tests..."
                    sh 'mvn test' 
                }
            }
        }

        // stage('Static Analysis (SonarQube)') {
        //     steps {
        //         script {
        //             echo "Running SonarQube analysis..."
        //             withSonarQubeEnv('sonarqube-server') {
        //                 sh 'mvn sonar:sonar'
        //             }
        //         }
        //     }
        // }

        stage('Push to Docker Hub') {
            steps {
                script {
                    sh '''
                    echo "${DOCKER_HUB_CREDENTIALS_PSW}" | docker login -u "${DOCKER_HUB_CREDENTIALS_USR}" --password-stdin
                    docker push benjaminssempala/petclinic2:latest
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Build and Push successful!"
        }
        failure {
            echo "Build failed."
        }
    }
}
