pipeline {
  agent any

  environment {
    DOCKER_HUB_CREDENTIALS = credentials('docker-hub')
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/BenjaminSsempala/spring-petclinic.git'
      }
    }

    /*
     * BUILD IMAGE USING KANIKO IN KUBERNETES
     */
    stage('Build Image') {
      agent {
        kubernetes {
          yaml ""
          "
          apiVersion: v1
          kind: Pod
          spec:
            containers:
            -name: kaniko
          image: gcr.io / kaniko - project / executor: latest
          command:
            -/busybox/cat
          tty: true ""
          "
        }
      }
      steps {
        container('kaniko') {
          sh ''
          ' /
          kaniko / executor\
            --context `pwd`\
            --dockerfile `pwd` / Dockerfile\
            --destination benjaminssempala / petclinic2: latest ''
          '
        }
      }
    }

    stage('Test') {
      steps {
        sh 'mvn test'
      }
    }

    stage('Push to Docker Hub') {
      steps {
        script {
          sh ""
          "
          echo "${DOCKER_HUB_CREDENTIALS_PSW}" | docker login\ -
            u "${DOCKER_HUB_CREDENTIALS_USR}"--password - stdin

          docker push benjaminssempala / petclinic2: latest ""
          "
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