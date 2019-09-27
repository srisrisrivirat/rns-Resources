def mvnHOME
def remote = [:]
    	remote.name = 'deploy'
    	remote.host = '192.168.33.20'
    	remote.user = 'root'
    	remote.password = 'vagrant'
    	remote.allowAnyHosts = true
pipeline {
	agent none 
	stages {
	//def mvnHOME
	stage ('Preparation') {
	agent {
	label 'Slave'
	}
	steps {
	git 'https://github.com/srisrisrivirat/Maven-Java-Project.git'
	stash 'Source'
	script {
	mvnHOME = tool 'LocalMaven'
	}
	}
	}
	stage ('Static-Analysis') {
	agent {
	label 'Slave'
	}
	steps {
	sh "'${mvnHOME}/bin/mvn' clean cobertura:cobertura"
	}
	post {
	succes {
	cobertura autoUpdateHealth: false, autoUpdateStability: false, coberturaReportFile: 'target/site/cobertura/coverage.xml', conditionalCoverageTargets: '70, 0, 0', failUnhealthy: false, failUnstable: false, lineCoverageTargets: '80, 0, 0', maxNumberOfBuilds: 0, methodCoverageTargets: '80, 0, 0', onlyStable: false, sourceEncoding: 'ASCII', zoomCoverageChart: false
	}
	}
	}
	stage ('Build') {
	agent {
	label 'Slave'
	}
	steps {
	sh "'${mvnHome}/bin/mvn' clean package"	
	}
	post {
	always {
	junit 'target/surefire-reports/*.xml'
    archiveArtifacts '**/*.war'
    fingerprint '**/*.war'
	}
	}
	}
	stage ('Deploy-to-Stage') {
	agent {
	label 'Slave'
	}
	//SSH-Steps-Plugin should be installed
    //SCP-Publisher Plugin (Optional)
	steps {
	//sshScript remote: remote, script: "abc.sh"
	sshPut remote: remote, from: 'target/*.war', into: '/workspace/appServer/webapps'

	}
	}
	stage ('Integration-test') {
	agent {
	label 'Slave'
	}
	steps {
	parallel (
	'integration': {
	untash 'Source'
	sh "'${mvnHOME}/bin/mvn' clean verify"
	}, 'quality': {
	unstash 'Source'
	sh "'${mvnHome}/bin/mvn' clean test"
	}
	)
	}
	}
	stage ('Approve') {
	agent {
	label 'Slave'
	}
	steps {
	timeout(time: 7, unit: 'DAYS') {
	input message: 'Do you want to deploy?', submitter: 'admin'
	}
	}
	}
	stage ('Prod-Deploy') {
	agent {
	label 'Slave'
	}
	steps {
	unstash 'Source'
	sh "'${mvnHome}/bin/mvn' clean package"
	}
	post {
	always {
	archiveArtifacts '**/*.war'
	}
	}
	}

	}
}
