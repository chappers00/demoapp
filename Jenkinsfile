node('ssh-agent') {
 stage('Checkout code') {
     checkout scm
 }
 stage('Build') {
 def mvnHome = tool 'maven3'
 sh "${mvnHome}/bin/mvn package"
 }
}