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
}

     