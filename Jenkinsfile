def DOCKER_IMG_BASENAME='demo-app'

node('ssh-agent') {
	stage('Checkout code') {
		checkout scm
	}
	stage('Build') {
		def mvnHome = tool 'maven3'
		sh "${mvnHome}/bin/mvn package"
		
		// Stash is like archiveArtifacts,
		// But is only valid for the build duration
		stash includes: '**/target/*.jar',
			name: 'application-binaries'
	}
}

node('docker') {
	stage('Docker Build') {
		//We need Dockerfile in the source code
		checkout scm
		
		//And the compiled application
		unstash 'application-binaries'
		sh "docker build -t ${DOCKER_IMG_BASENAME}:latest ./"
	}
}