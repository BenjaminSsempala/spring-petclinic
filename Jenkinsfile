pipeline {
    // 1. DEFINE A KUBERNETES AGENT POD
    // (These Groovy comments are fine)
    agent {
        kubernetes {
            // Define the pod structure using YAML
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: jenkins/maven:latest
    command: ['sleep']
    args: ['999999'] # <-- FIXED: Was '//'
    tty: true
  - name: kaniko
    # We use the debug tag because it has a shell
    image: gcr.io/kaniko-project/executor:latest-debug
    command: ['cat'] # <-- FIXED: Was '//'
    tty: true
"""
            // The 'jnlp' (maven) container will be used for all
            // steps by default, unless we specify another.
            defaultContainer 'jnlp'
        }
    }

    environment {
        // We will use these credentials in the Kaniko stage
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub')
    }

    stages {
        stage('Checkout') {
            // This runs in the default 'jnlp' container
            steps {
                git branch: 'main', url: 'https://github.com/BenjaminSsempala/spring-petclinic.git'
            }
        }

        stage('Test') {
            // This also runs in the default 'jnlp' (maven) container
            steps {
                script {
                    echo "Running tests..."
                    sh 'mvn test'
                }
            }
        }

        // 2. THIS IS THE NEW BUILD STAGE
        stage('Build and Push with Kaniko') {
            // 3. Tell Jenkins to run these steps in the 'kaniko' container
            steps {
                container('kaniko') {
                    script {
                        sh '''
                        echo "🏗️ Building and pushing image with Kaniko..."
                        
                        # 4. Create the Docker config.json for Kaniko
                        # This tells Kaniko how to log in to Docker Hub
                        echo "Creating Docker config..."
                        AUTH=$(echo -n "${DOCKER_HUB_CREDENTIALS_USR}:${DOCKER_HUB_CREDENTIALS_PSW}" | base64)
                        mkdir -p /kaniko/.docker
                        echo "{\"auths\":{\"https://index.docker.io/v1/\":{\"auth\":\"${AUTH}\"}}}" > /kaniko/.docker/config.json
                        
                        # 5. Run the Kaniko executor
                        # It reads the Dockerfile, builds, and pushes
                        /kaniko/executor \
                            --dockerfile=Dockerfile \
                            --context=dir://. \
                            --destination=benjaminssempala/petclinic2:latest
                        
                        echo "✅ Kaniko build complete!"
                        '''
                    }
                }
            }
        }
    } // end stages

    post {
        success {
            echo "Build and Push successful!"
        }
        failure {
            echo "Build failed."
        }
    }
}