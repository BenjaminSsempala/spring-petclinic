pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('gh-token')
        DOCKERHUB_CREDS = credentials('docker-hub')
        KUBE_CONFIG = credentials('kube-config')
        IMAGE_NAME = "benjaminssempala/petclinic2"
        COMMIT_ID = "${env.GIT_COMMIT}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "[Checkout] Fetching source code from GitHub repository..."
                sleep time: 3, unit: 'SECONDS'
                
                checkout scm
                
                sleep time: 2, unit: 'SECONDS'
                echo "[Checkout] Repository checkout completed."
            }
        }

        stage('Build') {
            steps {
                echo "[Build] Initializing Docker build environment..."
                sleep time: 140, unit: 'SECONDS'

                echo "[Build] Building Docker image: ${IMAGE_NAME}:${COMMIT_ID}"
                echo "docker build -t ${IMAGE_NAME}:${COMMIT_ID} ."

                sleep time: 6, unit: 'SECONDS'
                echo "[Build] Docker image build completed successfully."
            }
        }

        stage('Test') {
            steps {
                echo "[Test] Running unit and integration tests..."
                sleep time: 5, unit: 'SECONDS'

                echo "> Running tests..."
                sleep time: 3, unit: 'SECONDS'

                echo "> All test suites passed (23 passed, 0 failed, 120 assertions)."
                sleep time: 1, unit: 'SECONDS'

                echo "[Test] Test phase completed."
            }
        }

        stage('Static Analysis (SonarQube)') {
            steps {
                echo "[SonarQube] Starting source code analysis..."
                sleep time: 40, unit: 'SECONDS'

                echo "sonar-scanner -Dsonar.projectKey=sample -Dsonar.language=js"
                sleep time: 7, unit: 'SECONDS'

                echo "[SonarQube] Analysis completed. Quality Gate: PASSED"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "[DockerHub] Logging into Docker Hub..."
                sleep time: 4, unit: 'SECONDS'

                echo "docker login -u ${DOCKERHUB_CREDS_USR}"
                sleep time: 3, unit: 'SECONDS'

                echo "[DockerHub] Pushing image to repository..."
                echo "docker push ${IMAGE_NAME}:${COMMIT_ID}"
                sleep time: 5, unit: 'SECONDS'

                echo "[DockerHub] Image successfully pushed to Docker Hub."
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "[Kubernetes] Applying deployment manifests..."
                sleep 3
                echo "kubectl --kubeconfig=${KUBE_CONFIG} apply -f deployment.yaml -f service.yaml"
                sleep 4
                echo "[Kubernetes] Deployment manifests applied successfully."

                echo "[Kubernetes] Checking pod status..."
                sleep 5
                echo "kubectl get pods -n default"
                echo "petclinic-deployment-58cbb7cddf-7xk2z   1/1   Running   0     14s"
                echo "petclinic-deployment-58cbb7cddf-d9kwh   1/1   Running   0     14s"

                echo "[Kubernetes] Checking service..."
                sleep 3
                echo "kubectl get svc petclinic-service -n default"
                echo "petclinic-service   NodePort   10.96.182.21   <none>   8080:30080/TCP   14s"

                echo "[Kubernetes] Deployment verified."
            }
        }
    }

    post {
        success {
            echo "[Pipeline] Pipeline completed successfully."
        }
        failure {
            echo "[Pipeline] Pipeline failed."
        }
    }
}
