node {
	properties([
		[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', numToKeepStr: '15']]
	])
	
	stage('Checkout') {
		checkout scm
	}
	
	stage('Gradle Build') {
		sh "./gradlew clean cleanGenerateXtext build createLocalMavenRepo -PuseJenkinsSnapshots=true -PcompileXtend=true -PignoreTestFailures=true --refresh-dependencies --continue"
		step([$class: 'JUnitResultArchiver', testResults: '**/build/test-results/test/*.xml'])
	}
	
	def workspace = pwd()
	def mvnHome = tool 'M3'
	env.M2_HOME = "${mvnHome}"
	dir('.m2/repository/org/eclipse/xtext') { deleteDir() }
	dir('.m2/repository/org/eclipse/xtend') { deleteDir() }
	
	stage('Maven Plugin Build') {
		sh "${mvnHome}/bin/mvn -f maven-pom.xml --batch-mode --update-snapshots -fae -PuseJenkinsSnapshots -Dmaven.test.failure.ignore=true -Dmaven.repo.local=${workspace}/.m2/repository clean deploy"
	}
	
	stage('Maven Tycho Build') {
		wrap([$class:'Xvnc', useXauthority: true]) {
			sh "${mvnHome}/bin/mvn -f tycho-pom.xml --batch-mode -fae -Dmaven.test.failure.ignore=true -Dmaven.repo.local=${workspace}/.m2/repository clean install"
		}
		step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/*.xml'])
	}
	
	stage('Gradle Longrunning Tests') {
		sh "./gradlew longrunningTest -PuseJenkinsSnapshots=true -PignoreTestFailures=true --continue"
		step([$class: 'JUnitResultArchiver', testResults: '**/build/test-results/longrunningTest/*.xml'])
	}

		stage('Maven Tycho Build (Latest)') {
		wrap([$class:'Xvnc', useXauthority: true]) {
			sh "${mvnHome}/bin/mvn -f tycho-pom.xml -PuseLatestTarget --batch-mode -fae -Dmaven.test.failure.ignore=true -Dmaven.repo.local=${workspace}/.m2/repository2 clean install"
		}
		step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/*.xml'])
	}
	
	archive 'build/**'
}
