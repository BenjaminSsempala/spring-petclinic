pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub')
        GITHUB_TOKEN = credentials('github-token')
        KUBECONFIG = credentials('kubeconfig')
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/BenjaminSsempala/spring-petclinic.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t benjaminssempala/petclinic2:latest .
                    docker login -u ${DOCKER_HUB_CREDENTIALS_USR} -p ${DOCKER_HUB_CREDENTIALS_PSW}
                    docker push benjaminssempala/petclinic2:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes.') {
            steps { 
                script {
                    writeFile file: 'kubeconfig', text: "${KUBECONFIG}"
                    withEnv(["KUBECONFIG=${WORKSPACE}/kubeconfig"]) {
                        sh """
                        kubectl apply -f k8s/deployment.yaml
                        kubectl rollout status deployment/<your-app-deployment> -n <your-namespace>
                        """
                    }
                }
            }
        }
    }
}
