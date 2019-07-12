node {
 stage('Checkout code') {
     git 'http://localhost:5000/gitserver/butler/demoapp.git'
 }
 stage('Build') {
 def mvnHome = tool 'maven3'
 sh "${mvnHome}/bin/mvn package"
 }
}