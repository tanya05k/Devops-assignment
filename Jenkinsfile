pipeline {
  agent any

  environment {
      IMAGE_NAME = "skdoc2026/node-devops"
  }

  stages {

   stage('Checkout') {
  steps {
    checkout scm
  }
}

    stage('Build Docker Image') {
  steps {
    sh 'docker build -t $IMAGE_NAME ./app'
  }
}

    stage('Trivy Security Scan') {
      steps {
        sh 'trivy image $IMAGE_NAME'
      }
    }

    stage('Docker Login') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {

          sh '''
          echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
          '''
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        sh 'docker push $IMAGE_NAME'
      }
    }

    stage('Terraform Deploy') {
  steps {
    withCredentials([
      string(credentialsId: 'aws-access-key', variable: 'AWS_ACCESS_KEY_ID'),
      string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_ACCESS_KEY')
    ]) {
      sh '''
      echo "Checking AWS environment variables..."
      env | grep AWS

      cd terraform
      terraform init
      terraform apply -auto-approve
      '''
    }
  }
}
  }
}