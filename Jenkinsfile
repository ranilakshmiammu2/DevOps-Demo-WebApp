pipeline {
  agent any
  stages {
    stage('init') {
      steps {
        sh 'echo "Squad #8 Pipeline"'
      }
    }

    stage('Build Web App') {
      steps {
        echo 'Build Web App'
        withMaven(maven: 'Maven3.6.3') {
          sh 'mvn compile'
        }

      }
    }

  }
  environment {
    SONAR_CREDS = credentials('sonar-user-pass')
  }
}