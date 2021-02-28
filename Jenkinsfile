pipeline {
  agent any
  stages {
    stage('init') {
      steps {
        sh 'echo "Squad #8 Pipeline"'
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
