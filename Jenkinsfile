
node {
    try{
	def mvnHome
	def server =Artifactory.server 'anoop'
	stage('Preparation') { 
		git 'https://github.com/anoop600/durga.git'
		mvnHome = tool 'Maven'
	}
	stage('SonarQube analysis') {
		withSonarQubeEnv('anoop-azure') {
			sh 'mvn clean package sonar:sonar'
		}
	}
   
	stage("Quality Gate") {
        timeout(time: 1, unit: 'HOURS') { 
			def qg = waitForQualityGate() 
			if (qg.status != 'OK') {
				error "Pipeline aborted due to quality gate failure: ${qg.status}"
				mail bcc: '', body: 'Build Failed', cc: '', from: '', replyTo: '', subject: 'Build Failed', to: 'anoop.jain10@gmail.com'
				currentBuild.status='FAILURE'
			}
		}
	}

	stage('Build') {
        sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"    
	}
	
	stage('Artifactory upload') {
        def uploadSpec = """{
		"files": [ { "pattern": "/var/lib/jenkins/workspace/DemoCapability/target/*.war", "target": "srini" } ] }""" 
            server.upload(uploadSpec) 
       
        }
        
    stage('downloading artifact'){ 
		def downloadSpec="""{ "files":[ { "pattern":"srini/z12345.war", "target":"/var/lib/jenkins/workspace/DemoCapability/" } ] }""" 
		server.download(downloadSpec)
    }    
}catch(err){
    mail bcc: '', body: 'Build Failed', cc: '', from: '', replyTo: '', subject: 'Build Failed', to: 'anoop.jain10@gmail.com'
    
	stage('JIRA'){
		withEnv(['JIRA_SITE=anoop-jira']) {
			def testIssue = [fields: [ project: [key: 'TES'],
			summary: "Issue ${BUILD_NUMBER} ${env.JOB_NAME} ",
			description: 'BUG in CODE',
			issuetype: ["name":"Bug"]]]
			response = jiraNewIssue issue: testIssue
			echo response.successful.toString()
			echo response.data.toString()
		}
    }
    
    stage('comment'){
        withEnv(['JIRA_SITE=anoop-jira']) {
			jiraComment body: 'Build sucess', issueKey: 'TES-2'
			jiraAssignIssue idOrKey: 'TES-2', userName: 'admin'
		}
    }
    
    stage('JIRA-transition') {
      withEnv(['JIRA_SITE=anoop-jira']) {
        def transitionInput =
        [
          transition: [
            id: '41'			  
          ]
        ]
        jiraTransitionIssue idOrKey: 'TES-2', input: transitionInput
      }
    }
    currentBuild.result = 'FAILURE'
    }
}
