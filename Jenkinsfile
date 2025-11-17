pipeline {
    agent any

    environment {
        GITHUB_TOKEN      = credentials('gh-token')
        DOCKERHUB_CREDS   = credentials('docker-hub')
        KUBECONFIG_CRED   = credentials('kube-config')
        IMAGE_NAME        = "benjaminssempala/petclinic"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/BenjaminSsempala/spring-petclinic.git'
            }
        }

        stage('Build') {
            steps {
                echo "[Build] Compiling application..."
                sh "mvn clean package -DskipTests"
                echo "[Build] Build completed."
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    docker build -t benjaminssempala/petclinic:latest .
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                echo "[Test] Running tests"
                sh "mvn test"
                echo "[Test] Tests passed successfully."
            }
        }

        stage('Static Analysis (SonarQube)') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=petclinic \
                        -Dsonar.projectName=petclinic \
                        -Dsonar.java.binaries=target/
                    """
                }
                echo "[SonarQube] Analysis completed."
            }
        }

        stage('Push Image') {
            steps {
                echo "[DockerHub] Logging in..."
                sh "echo ${DOCKERHUB_CREDS_PSW} | docker login -u ${DOCKERHUB_CREDS_USR} --password-stdin"

                echo "[DockerHub] Pushing image..."
                sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"

                echo "[DockerHub] Push completed."
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Applying Kubernetes manifests..."

                withCredentials([file(credentialsId: 'kube-config', variable: 'KUBECONFIG_FILE')]) {
                    sh """
                        export KUBECONFIG=${KUBECONFIG_FILE}

                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml

                        echo "[Deploy] Updating image..."
                        kubectl set image deployment/petclinic-app petclinic=${IMAGE_NAME}:${BUILD_NUMBER}
                    """
                }

                echo "Deployment applied successfully."
            }
        }

        stage('Verify Rollout') {
            steps {
                echo "[Verify] Checking rollout status..."

                withCredentials([file(credentialsId: 'kube-config', variable: 'KUBECONFIG_FILE')]) {
                    sh """
                        export KUBECONFIG=${KUBECONFIG_FILE}
                        kubectl rollout status deployment/petclinic-app
                    """
                }

                echo "[Verify] Rollout completed successfully."
            }
        }

    } 

    post {
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
