node {
	
// Clean workspace before doing anything
    deleteDir()

    try {
        stage ('Clone') {
        	checkout scm
        }
        stage ('Build') {
        	sh "ansible-playbook ansible-04-mission/netdevops_dev.yaml"
         }
        stage ('Notification') {
                    sh "ansible-playbook ansible-04-mission/notification-dev.yaml"
       }
        stage ('MergetoMaster') {
                     sh (script: "cd /var/lib/jenkins/workspace/pipeline && git checkout master")
                     sh (script: "cd /var/lib/jenkins/workspace/pipeline && git branch -u origin/master")
                     sh (script: "cd /var/lib/jenkins/workspace/pipeline && git merge origin/dev")
		     sh (script: "cd /var/lib/jenkins/workspace/pipeline && git push http://netdevopsuser:network@198.18.134.48:3000/netdevopsuser/netdevops-ansible")        
         }

        stage ('Configure Production'){
                    sh "ansible-playbook ansible-04-mission/netdevops_prod.yaml"
      }
        stage ('Notification') {
                    sh "ansible-playbook ansible-04-mission/notification-prod.yaml"
       }
     stage ('Notification Details') {
            sparkSend credentialsId: 'ac1a9013-e1e7-48fc-be57-eee6b8d5f9cb', message: '[$JOB_NAME]($BUILD_URL)', messageType: 'text', spaceList: [[spaceId: 'Y2lzY29zcGFyazovL3VzL1JPT00vZTg4Y2M4NjAtMTA0ZS0zMmRjLWEzY2YtMTcxNTcwNzRkMGMw', spaceName: 'NetDevOps CICD Bot']] 
         }


    } catch (err) {
        currentBuild.result = 'FAILED'
        throw err
    }
}




