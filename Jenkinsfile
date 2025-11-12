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

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    docker build -t benjaminssempala/petclinic2:latest .
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
