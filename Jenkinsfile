/* This script is designed and maintained by Ganesh Palnitkar
*/
def notify(status) {
  mail (
        body:"""${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':
                 Check console output at,
                 href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]""",
        cc: '<nilesh.zend@gmail.com>',
        subject: """JenkinsNotification: ${status}:""",
        to: '<nilesh.zend@outlook.com>'
       )
 }
def server = Artifactory.server 'artifactory'
def uploadSpec = """{
  "files": [
	{
	  "pattern": "target/petclinic.war",
	  "target": "petclinicRepo/"
	}
		   ]
  }"""

/*def downloadSpec = """{
 "files": [
  {
      "pattern": "petclinicRepo/*.war",
      "target": "./"
    }
 ]
}"""
*/
pipeline {
 agent none
    stages {
      stage('SCM_Chekout') {
          agent { label "master" }
			steps {
			    script {
					notify('build-started')
				}
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/master']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    submoduleCfg: [],
                    userRemoteConfigs: [[url: 'https://github.com/nileshzend/Maven-petclinic-project.git']]])
            }
        }
      stage('Build'){
          agent { label "master" }
			steps {
                sh 'mvn -f pom.xml clean package'
            }
        }

	  stage('push-to-artifactory') {
          agent { label "master" }
			steps {
                script {
				   server.upload(uploadSpec)
            }
        }
	}
	  stage('Deploy') {
          agent { label "Tomcat" }
			steps {
			    script {
				  input('Deploy Package to Production?')
				  notify('Deployment-to-Production')
				}
					sh 'wget http://172.28.128.5:8081/artifactory/petclinicRepo/petaclinic.war'
					sh 'cp ./.war /opt/tomcat/webapps/'
					sh '/opt/tomcat/bin/catalina.sh run'

            }
         }
    }
}
