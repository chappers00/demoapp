def DOCKER_IMG_BASENAME='demo-app'
def GIT_SHORT_CHANGESET='latest'

node('ssh-agent') {
	stage('Checkout code') {
		checkout scm
		
		//We can set a groovy variable from a "sh" command stdout.
		//Do not forget to remove endlines with trim()
		GIT_SHORT_CHANGESET =
			sh(returnStdout:true, script: 'git rev-parse --short HEAD').trim()
	}
	withEnv(["PATH+MAVEN=${tool 'maven3'}/bin"]) {
		stage('Build') {
			sh "mvn package"
			junit '**/target/surefire-reports/*.xml'
			
			// Stash is like archiveArtifacts,
			// But is only valid for the build duration
			stash includes: '**/target/*.jar',
				name: 'application-binaries'
		}
		stage('Integration Tests') {
			sh "mvn verify -fn"
			junit '**/target/failsafe-reports/*.xml'
	}
}

node('docker') {
	stage('Docker Build') {
		//We need Dockerfile in the source code
		checkout scm
		
		//And the compiled application
		unstash 'application-binaries'
		sh "docker build -t ${DOCKER_IMG_BASENAME}:${GIT_SHORT_CHANGESET} ./"
	}
}