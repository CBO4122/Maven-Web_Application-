pipeline {
    agent any
    tools {
        maven "maven_3.9.10"
    }
    triggers {
           pollSCM('* * * * *')
         }
    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/CBO4122/Maven-Web_Application-.git'
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
                    curl -u tomcat:tomcat \\
                    --upload-file ${env.WORKSPACE}/target/maven-web-application.war \\
                    "http://35.154.51.226:8080/manager/text/deploy?path=/maven-web-application&update=true"
                """
            }
        }
    }

    post {
        success {
            notifyBuild(currentBuild.result)
        }
        failure {
            notifyBuild(currentBuild.result)
        }
    }
}

def notifyBuild(String buildStatus = 'STARTED') {
    buildStatus = buildStatus ?: 'SUCCESS'

    def colorCode = '#FF0000'
    def summary = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"

    if (buildStatus == 'STARTED') {
        colorCode = '#FFFF00'
    } else if (buildStatus == 'SUCCESS') {
        colorCode = '#00FF00'
    }

    // Send Slack notification (make sure Slack is configured in Jenkins)
    slackSend(color: colorCode, message: summary, channel: '#cbo_devops')
}
