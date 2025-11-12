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
            agent {
                docker {
                    // Use the Docker CLI image with Docker-in-Docker enabled
                    image 'docker:24.0-dind'
                    args '--privileged'
                    reuseNode true   // keep the same workspace mounted
                }
            }
            steps {
                script {
                    // Use the Docker Pipeline plugin to build and push
                    def customImage = docker.build("benjaminssempala/petclinic2:${env.BUILD_ID}", ".")
                    
                    // Optional: run commands inside the image
                    customImage.inside {
                        echo "Running tests inside Docker image..."
                        sh 'java -version'
                        sh 'mvn test'
                    }
                    
                    // Push to Docker Hub
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub') {
                        customImage.push()           // push build tag
                        customImage.push('latest')   // push latest tag
                    }
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
