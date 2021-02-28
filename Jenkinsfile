pipeline {
  agent any
  tools{
        maven 'Maven3.6.3' 
    }
  environment {
        SONAR_ADMIN_CRED = credentials('sonar')
    }
  stages {
    stage('init') {
      steps {
        sh 'echo "Squad #8 Pipeline"'
      }
    }
  stage('SourceCode-Checkhout'){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/devopsquad8/DevOps-Demo-WebApp/']]])
            }
        }
        
  stage('Code-Analysis'){
            steps{
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn $SONAR_MAVEN_GOAL -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=$SONAR_ADMIN_CRED_USR -Dsonar.password=$SONAR_ADMIN_CRED_PSW sonar:sonar -f pom.xml'
                }
            }
        }
  stage('Build Artifact'){
            steps{
                sh 'mvn clean install -f pom.xml'
            }
            
        }
        
   stage('Upload-to-Artifactory'){
            steps{
                
                rtUpload (
                    serverId: 'artifactory',
                    spec: """{
                            "files": [
                                    {
                                        "pattern": "target/*.war",
                                        "target": "libs-release-local"
                                    }
                                ]
                            }"""
                )
                
                rtPublishBuildInfo (
                    serverId: 'artifactory'
                )
            }
        }
        
    stage('Deploy-to-QA'){
            steps{
                deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://18.220.112.107:8080')], contextPath: '/QAWebapp', onFailure: false, war: '**/*.war'
            }
            
        }
  }
}
