pipeline {
  agent any
  
  tools {
    maven 'Maven3.6.3'
  }

  environment {
    SONAR_CREDS = credentials('sonar-user-pass')
  }

  stages {
    stage('init') {
      steps {
        echo "Squad #8 Pipeline"
      }
    }

    stage('Static-Code Analysis') {
      steps {
        echo 'Static-Code Analysis'
        withSonarQubeEnv(credentialsId: 'sonar2', installationName: 'sonarqube') {
            sh "mvn sonar:sonar -Dsonar.login=$SONAR_CREDS_USR -Dsonar.password=$SONAR_CREDS_PSW -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java"
            sh "mvn validate"
        }
        timeout(time: 1, unit: 'HOURS') { // pipeline will be terminated if a timeout occurs
	    def qg = waitForQualityGate() // Reuse taskId  collected by withSonarQubeEnv
	    if (qg.status != 'OK') {
	      error "Pipeline aborted.Quality gate failure: ${qg.status}"
	    }
	}             
      }
    }

    stage('Build Web App') {
      steps {
        echo 'Build Web App'
        sh 'mvn compile'
      }
    }

    stage('Build Artifact') {
      steps {
        sh 'mvn clean install -f pom.xml'
      }

    }

    stage('Upload-to-Artifactory') {
      steps {

        rtUpload(
          serverId: 'artifactory',
          spec: """{
              "files": [{
                "pattern": "target/*.war",
                "target": "libs-release-local"
              }]
            }
            """
        )

          rtPublishBuildInfo(
            serverId: 'artifactory'
          )
      }
    }
  
    stage('Deploy-to-QA') {
      steps {
        deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://172.31.87.233:8080')], contextPath: '/QAWebapp', onFailure: false, war: '**/*.war'
      }

    }

    stage('Slack-Notification') {
      steps {
        slackSend channel: 'alerts', message: "Deployed to QA: Job - '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
      }
    }

    stage('Testing') {
      steps {
        parallel(
          UITest: {
            echo "UI Test"
            sh 'mvn test -f functionaltest/pom.xml'
            publishHTML(target: [allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test Report', reportTitles: 'UI Test Report'])
          },
          PerformanceTest: {
            echo "Performance test"
            //blazeMeterTest credentialsId: 'blazeMeter', testId: ' 9015188.taurus', workspaceId: '755418'
          }
        )
      }
    }

    stage('Deploy-to-PROD') {
      steps {
        deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://172.31.92.140:8080/')], contextPath: '/ProdWebapp', onFailure: false, war: '**/*.war'
      }

    }

    stage('Slack-Notification-Prod') {
      steps {
        slackSend channel: 'alerts', message: "Deployed to PROD: Job - '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"
      }
    }

    stage('Sanity-Test') {
      steps {
        echo "Sanity-Test"
        sh 'mvn test -f Acceptancetest/pom.xml'
        publishHTML(target: [allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'SanityTestReport', reportTitles: 'Sanity Test Report'])
      }
      
    }

  }
}
