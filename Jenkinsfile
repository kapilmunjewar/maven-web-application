node{
    
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
sshagent(['5ac71c4c-93e0-44f8-a464-86880b1ecbc5']) {
    sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@13.126.76.233:/opt/apache-tomcat-9.0.63/webapps/"
}
}


}
