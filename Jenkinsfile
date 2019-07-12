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

//Putting "input" inside a stage will allow
//The GUI to visualise it and show the popup
//If not you will have to go to the console output of the build
stage('Waiting approval') {
	//We do not put "Input" within a node
	//To avoid eating an executor while waiting
	timeout(time:1, unit:'DAYS') {
		input message: "Is it OK to deploy docker image?", ok:'Deploy'
	}
}

//We do not put "build witin a node job for the same reasons as "input"
stage('Deploy') {
	//This will call the downstream job and block until it finishes
	//Unless disabled, downstream job status will be reported to the pipeline
	build job: 'demoapp-staging-deployer',
		parameters: [string(name:'DOCKER_IMAGE',
		value: "${DOCKER_IMG_BASENAME}:${GIT_SHORT_CHANGESET}")]
}