pipeline {
    agent any
    tools{
           maven "maven_3.9.10"
         }

    stages {
        stage('git checkout') {
            steps {
                  'https://github.com/CBO4122/Maven-Web_Application-.git'
            }
        }
        stage('Compile') {
            steps {
                  sh 'mvn clean compile'
            }
        }
        stage('Build') {
            steps {
                  sh 'mvn clean package'
            }
        }
        stage('SonarQube Report') {
            steps {
                  sh 'mvn sonar:sonar'
            }
        }
        stage('Deploy to Nexus') {
            steps {
                  sh 'mvn clean deploy'
            }
        }
        stage('Deploy to Tomcat') {
            steps {

               sh """

            curl -u tomcat:tomcat \
            --upload-file /var/lib/jenkins/workspace/jio-Declarative-way-PL/target/maven-web-application.war \
            "http://35.154.51.226:8080/manager/text/deploy?path=/maven-web-application&update=true"
          
                """   
            }
        }

    } // stages closing

    post { 
    success {
            notifyBuild(currentBuild.result)
            }
    failure {
            notifyBuild(currentBuild.result)
            }
    }
} // pipeline end

def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary, channel: '#cbo_devops')
}
