pipeline {
    agent any

    tools {
        maven "maven_3.9.10"
    }

    triggers {
        pollSCM('* * * * *')  // Polls SCM every minute – adjust if needed
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'UAT', url: 'https://github.com/CBO4122/Maven-Web_Application-.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }

        stage('SonarQube Report') {
            steps {
                sh 'mvn sonar:sonar'
            }
        }

        stage('Deploy to Nexus') {
            steps {
                sh 'mvn deploy'
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
            script {
                notifyBuild('SUCCESS')
            }
        }
        failure {
            script {
                notifyBuild('FAILURE')
            }
        }
    }
}

def notifyBuild(String buildStatus = 'STARTED') {
    buildStatus = buildStatus ?: 'SUCCESS'

    def colorCode = '#FF0000'  // Default RED for failure
    if (buildStatus == 'STARTED') {
        colorCode = '#FFFF00'  // Yellow
    } else if (buildStatus == 'SUCCESS') {
        colorCode = '#00FF00'  // Green
    }

    def summary = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})"

    // Slack send: Make sure slack plugin and credentials are configured
    slackSend(color: colorCode, message: summary, channel: '#cbo_devops')
}
