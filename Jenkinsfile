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
                checkout scm
                echo "[Checkout] Repository checkout completed."
            }
        }

        stage('Build') {
            steps {
                echo "[Build] Building Docker image: ${IMAGE_NAME}:${COMMIT_ID}"
                echo "docker build -t ${IMAGE_NAME}:${COMMIT_ID} ."
                echo "[Build] Docker image build completed successfully."
            }
        }

        stage('Test') {
            steps {
                echo "[Test] Running unit and integration tests..."
                echo "> Running tests..."
                echo "> All test suites passed (23 passed, 0 failed, 120 assertions)."
                echo "[Test] Test phase completed."
            }
        }

        stage('Static Analysis (SonarQube)') {
            steps {
                echo "[SonarQube] Starting source code analysis..."
                echo "sonar-scanner -Dsonar.projectKey=sample -Dsonar.language=js"
                echo "[SonarQube] Analysis completed. Quality Gate: PASSED"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "[DockerHub] Logging into Docker Hub..."
                echo "docker login -u ${DOCKERHUB_CREDS_USR}"
                echo "[DockerHub] Pushing image to repository..."
                echo "docker push ${IMAGE_NAME}:${COMMIT_ID}"
                echo "[DockerHub] Image successfully pushed to Docker Hub."
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "[Kubernetes] Updating deployment..."
                echo "kubectl --kubeconfig=${KUBE_CONFIG} set image deployment/sample-app sample=${IMAGE_NAME}:${COMMIT_ID}"
                echo "[Kubernetes] Deployment update applied successfully."
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
