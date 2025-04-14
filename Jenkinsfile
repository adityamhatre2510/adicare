pipeline {
  agent any

  environment {
    IMAGE_NAME = "adibowvalley/adicare-hospital"
    CONTAINER_NAME = "adicare"
    EC2_IP = "98.84.168.188"
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
          sh '''#!/bin/bash
            echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
            docker push adibowvalley/adicare-hospital:latest
          '''
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
            sh '''#!/bin/bash
              ssh -o StrictHostKeyChecking=no -i "$KEY" "$USER@98.84.168.188" <<EOF
                docker stop adicare || true
                docker rm adicare || true
                docker pull adibowvalley/adicare-hospital:latest
                docker run -d --name adicare -p 80:80 adibowvalley/adicare-hospital:latest
EOF
            '''
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
