properties([
  parameters([
    choice(choices: ['Dev', 'Stage', 'Prod'], description: 'Please select target Environment ', name: 'SELECT_ENV')
			])
	])
	
  node {
     def myRepo, gitCommit, gitBranch, shortGitCommit, previousGitCommit
     stage('Checkout') 
     {
        try {
           myRepo = checkout scm
           gitCommit = myRepo.GIT_COMMIT
           gitBranch = myRepo.GIT_BRANCH
           shortGitCommit = "${gitCommit[0..10]}"
           previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)
        }
        catch(Exception e) {
           println "ERROR => Code Checkout failed, exiting..."
           throw e
        }
     }
	 
	 stage('Build') {
      try {
			withMaven(maven: 'maven_3_6_3') {
			sh """
			mvn clean install -Dmaven.test.skip=true
			"""
			}
        }
      catch (Exception e) {
        println "Failed to Maven Build - ${currentBuild.fullDisplayName}"
        throw e
      }
    }
	
	stage('Unit Test') {
      try {
	  		withMaven(maven: 'maven_3_6_3') {
			sh """
			mvn test -Dmaven.test.skip=false
			"""
			}
        }
        
      catch (Exception e) {
        println "Failed to test - ${currentBuild.fullDisplayName}"
        throw e
      }
    }
	
	stage('Sonar Analysis') {
      try {
           withSonarQubeEnv('sonarqube_7_5') {
           sh """ 
		   mvn -DBranch=${gitBranch} -Dsonar.branch=${gitBranch} -e -B sonar:sonar
		   sh """
        }
      }
      catch (Exception e) {
        println "Failed to Sonar - ${currentBuild.fullDisplayName}"
        throw e
      }
    }
	
	stage('Quality Gate') {
      try {
          withSonarQubeEnv('sonarqube_7_5') {
          timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK' && qg.status != 'WARN') {
                error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
             }
          }
      }
      catch (Exception e) {
        println "Failed to Sonar QG - ${currentBuild.fullDisplayName}"
        throw e
      }
    }
}


     