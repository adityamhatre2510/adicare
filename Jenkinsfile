pipeline {
  agent any

  environment {
    // Docker and Deployment Settings
    IMAGE_NAME = "adibowvalley/adicare-hospital"
    CONTAINER_NAME = "adicare-hospital"
    EC2_IP = "34.228.65.134"
    SSH_CREDENTIALS_ID = "ec2-deploy-key"
  }

  stages {

    stage('Checkout') {
      steps {
        echo "📦 Cloning source code from GitHub..."
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        echo "🐳 Building Docker image..."
        sh "docker build -t ${IMAGE_NAME}:latest ."
      }
    }

    stage('(Optional) Unit Tests') {
      steps {
        echo "✅ Running unit tests (add real test scripts here)..."
        // sh './run-tests.sh'
      }
    }

    stage('Push Docker Image') {
      steps {
        echo "🚀 Pushing image to Docker Hub..."
        withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh """
            echo $PASSWORD | docker login -u $USERNAME --password-stdin
            docker push ${IMAGE_NAME}:latest
          """
        }
      }
    }

    stage('Deploy to EC2') {
      steps {
        echo "⚙️ Deploying container to EC2..."
        script {
          withCredentials([sshUserPrivateKey(
            credentialsId: SSH_CREDENTIALS_ID,
            keyFileVariable: 'KEY',
            usernameVariable: 'USER'
          )]) {
            sh """
              ssh -o StrictHostKeyChecking=no -i $KEY $USER@$EC2_IP '
                docker stop ${CONTAINER_NAME} || true
                docker rm ${CONTAINER_NAME} || true
                docker run -d --name ${CONTAINER_NAME} -p 80:80 ${IMAGE_NAME}:latest
              '
            """
          }
        }
      }
    }

    stage('Security Scan with Nmap') {
      steps {
        echo "🔒 Running post-deploy Nmap scan..."
        sh "nmap -A -T4 -p 80 ${EC2_IP} -oN nmap-scan.txt"
        archiveArtifacts artifacts: 'nmap-scan.txt'
      }
    }
  }

  post {
    success {
      echo "✅ Build, deploy, and scan completed successfully!"
    }
    failure {
      echo "❌ Pipeline failed. Please check the logs."
    }
  }
}
