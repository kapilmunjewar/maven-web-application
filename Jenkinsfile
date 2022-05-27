node {
    try {
        properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), [$class: 'JobLocalConfiguration', changeReasonComment: ''], pipelineTriggers([pollSCM('* * * * *')])])

        echo "Build No: ${BUILD_NUMBER}"
        echo "Job Name: ${JOB_NAME}"
        echo "Node Name: ${NODE_NAME}"

        def mavenHome = tool name: 'maven-3.8.5'

        stage('CheckoutCode'){
        git branch: 'development', credentialsId: 'be971745-92fe-4032-b5b4-c8f5263850e3', url: 'https://github.com/kapilmunjewar/maven-web-application'
        }

        stage('Build'){
        sh " ${mavenHome}/bin/mvn clean package"
        }

        stage('SonarQube'){
        sh " ${mavenHome}/bin/mvn sonar:sonar"
        }

        stage('Nexus'){
        sh " ${mavenHome}/bin/mvn deploy"
        }

        stage('Tomcat'){
        sshagent(['96839567-9e66-4e4d-a5f5-4daca9f076ed']) {
            sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@13.126.76.233:/opt/apache-tomcat-9.0.63/webapps/"
        }
        }

  } catch (e) {
    // If there was an exception thrown, the build failed
    currentBuild.result = "FAILED"
    throw e
  } finally {
    // Success or failure, always send notifications
    notifyBuild(currentBuild.result)
  }
}

def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}
