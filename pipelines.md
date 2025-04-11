Jenkins Pipelines for Python + Java Script + SonarQube

pipeline {
  agent any
  environment {
    SONAR_TOKEN = credentials('sonar-token-id')
  }
  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/your-repo.git'
      }
    }
    stage('Install Dependencies') {
      steps {
        sh 'pip install -r requirements.txt'
        sh 'npm install'
      }
    }
    stage('Run Tests') {
      steps {
        sh 'pytest tests/'      // for Python
        sh 'npm test'           // for JS
      }
    }
    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarQubeServer') {
          sh """
            sonar-scanner \
              -Dsonar.projectKey=your-key \
              -Dsonar.sources=. \
              -Dsonar.host.url=$SONAR_HOST_URL \
              -Dsonar.login=$SONAR_TOKEN
          """
        }
      }
    }
  }
}
