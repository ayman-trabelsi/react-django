pipeline {
  agent { label 'agent1' }

  environment {
    DOCKERHUB_CREDS = credentials('dockerhub')
    GITHUB_TOKEN    = credentials('github-token')
    SONAR_TOKEN     = credentials('sonarqube_token')
    PROJECT_NAME    = 'project1'
  }

  stages {
    stage('Checkout Source') {
      steps {
        git url: 'https://github.com/paidra/react-django.git', branch: 'main', credentialsId: 'github-token'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarQubeScanner') {
          sh '''#!/bin/bash
            docker run --rm \
              --network jenkins_default \
              -v $(pwd):/usr/src \
              sonarsource/sonar-scanner-cli \
              -Dsonar.projectKey=$PROJECT_NAME \
              -Dsonar.sources=. \
              -Dsonar.language=py \
              -Dsonar.host.url=http://sonarqube:9000 \
              -Dsonar.token=$SONAR_TOKEN
          '''
        }
      }
    }

    stage('DockerHub Login') {
      steps {
        sh '''#!/bin/bash
          echo "$DOCKERHUB_CREDS_PSW" | docker login -u "$DOCKERHUB_CREDS_USR" --password-stdin
        '''
      }
    }

    stage('Remove Old Docker Images') {
      steps {
        script {
          def images = sh(script: "docker images -q ayman43/frontend-app", returnStdout: true).trim()
          if (images) {
            sh "docker rmi -f ${images}"
          } else {
            echo "No old docker images to remove."
          }
        }
      }
    }

    stage('Build Docker Images') {
      steps {
        sh '''#!/bin/bash
          docker build -t ayman43/frontend-app -f frontend/Dockerfile ./frontend
          docker build -t ayman43/backend-app -f backend/Dockerfile ./backend
        '''
      }
    }

    stage('Push Images to DockerHub') {
      steps {
        sh '''#!/bin/bash
          docker push ayman43/frontend-app
          docker push ayman43/backend-app
        '''
      }
    }

    stage('Deploy Application Stack') {
      steps {
        sh '''#!/bin/bash
          docker-compose down || true
          docker-compose up -d --build
        '''
      }
    }
  }

  post {
    always {
      echo 'Pipeline execution complete.'
    }
    failure {
      echo 'Pipeline failed!'
    }
  }
}
