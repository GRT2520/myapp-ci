pipeline {
  agent any
  environment {
    IMAGE = "ashenblade2520/myapp-ci"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build Image') {
      steps { sh 'docker build -t $IMAGE:${BUILD_NUMBER} -t $IMAGE:latest .' }
    }
    stage('Test') {
      steps { sh 'docker run --rm $IMAGE:${BUILD_NUMBER} npm test' }
    }
    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
            usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh '''
            echo $PASS | docker login -u $USER --password-stdin
            docker push $IMAGE:${BUILD_NUMBER}
            docker push $IMAGE:latest
          '''
        }
      }
    }
    stage('Deploy to AWS EC2') {
      steps {
        sshagent(credentials: ['ec2-ssh']) {
          sh '''
            ssh -o StrictHostKeyChecking=no ubuntu@34.224.165.116 "
              docker pull $IMAGE:${BUILD_NUMBER} &&
              (docker rm -f myapp || true) &&
              docker run -d --name myapp -p 80:3000 --restart unless-stopped $IMAGE:${BUILD_NUMBER}
            "
          '''
        }
      }
    }
  }
}