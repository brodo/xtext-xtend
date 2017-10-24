// Tell Jenkins how to build projects from this repository
node {
	try {
		properties([
			[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', numToKeepStr: '15']]
		])
		
		stage 'Checkout'
		checkout scm
		
		dir('build') {
			deleteDir()
		}
		
		stage 'Gradle Build'
		try {
			sh "./gradlew clean cleanGenerateXtext build createLocalMavenRepo -PuseJenkinsSnapshots=true -PcompileXtend=true --refresh-dependencies --continue"
			archive 'build/maven-repository/**/*.*'
		} finally {
			step([$class: 'JUnitResultArchiver', testResults: '**/build/test-results/test/*.xml'])
		}
		
		def workspace = pwd()
		
		stage 'Gradle Longrunning Tests'
		try {
			sh "./gradlew longrunningTest -PuseJenkinsSnapshots=true --continue"
		} finally {
			step([$class: 'JUnitResultArchiver', testResults: '**/build/test-results/longrunningTest/*.xml'])
		}
		
		if ('UNSTABLE' == currentBuild.result) {
			slackSend color: 'warning', message: "Build Unstable - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
		} else {
			slackSend color: 'good', message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
		}
		
	} catch (e) {
		// TODO catch interrupt error instead
		if ('ABORTED' == currentBuild.result) { 
			slackSend message: "Build Aborted - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
		} else {
			slackSend color: 'danger', message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
		}
		throw e
	}
}
