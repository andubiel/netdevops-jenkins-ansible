node {
 	// Clean workspace before doing anything
    deleteDir()

    try {
        stage ('Clone') {
        	checkout scm
        }
        stage ('Build') {
        	sh "echo 'shell scripts to build project...'"
        }
        stage ('MergetoMaster') {
	        parallel 'static': {
	            sh "echo 'shell scripts to run static tests...'"
	        },
	        'unit': {
	            sh "echo 'shell scripts to run unit tests...'"
	        },
	        'integration': {
	            sh "echo 'shell scripts to run integration tests...'"
	        }
        }
      	stage ('Notification') {
            sparkSend credentialsId: '26c7085b-dfb6-43a4-bb39-94701724e781', message: '[$JOB_NAME]($BUILD_URL)', messageType: 'text', spaceList: [[spaceId: 'Y2lzY29zcGFyazovL3VzL1JPT00vZDZmMTIzODQtYjA2Yy0zNzkzLWFhN2MtMjI1YWQyYjZlNTA0', spaceName: 'ACI NetDevOps Bot']]	

       }
    } catch (err) {
        currentBuild.result = 'FAILED'
        throw err
    }
}




