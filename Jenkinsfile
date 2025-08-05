stage('SonarQube Analysis') {
  steps {
    withSonarQubeEnv('SonarQubeScanner') {
      withCredentials([string(credentialsId: 'sonarqube_token', variable: 'SONAR_TOKEN')]) {
        // Using host network so Docker can reach SonarQube on localhost:9000
        sh '''#!/bin/bash
          docker run --rm \
            --network host \
            -v $(pwd):/usr/src \
            sonarsource/sonar-scanner-cli \
            -Dsonar.projectKey=project1 \
            -Dsonar.sources=. \
            -Dsonar.language=py \
            -Dsonar.host.url=http://localhost:9000 \
            -Dsonar.token=$SONAR_TOKEN
        '''
      }
    }
  }
}

