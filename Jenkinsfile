pipeline {
    agent any
    
    stages {
        stage('Stage: SCM') { 

            steps {
                // Get some code from a GitLab repository
        git branch: 'developer-branch', credentialsId: 'gitlab-access', url: 'https://gitlab.com/clinistats/ribhus.git'
            }
        }
    stage('Stage: Maven Build'){

            steps {
                //Generate Artifact through maven build
                
            sh "mvn -f server/pom.xml clean install -DskipTests"
                  
        }
    }
   stage('Stage: upload to nexus') {
         
            steps {
                script {
                // publish the artact into Nexus repository
                def mavenPom = readMavenPom file: 'server/pom.xml'
  
    nexusArtifactUploader artifacts: [
	[
		artifactId: 'ribhus',
		classifier: '',
		file: "server/target/ribhus-${mavenPom.version}.jar",
		type: 'jar'
		]
	],
    credentialsId: 'nexus-access',
    groupId: 'dev',
    nexusUrl: "${NEXUS_HOMEPAGE}",
    nexusVersion: 'nexus3',
    protocol: 'http',
    repository: 'clinistats-repo',
    version: "${mavenPom.version}"
            }
            }
        }

    stage('Stage: Deploy into web/app server') {
       
            steps {
                script {
                def remote = [:]
        remote.name = "${Ribhus_Dev_Host}"
        remote.host = "${Ribhus_Dev_IP}"
        remote.user = "${remote_user}"
        remote.password = "${remote_pwd}"
        remote.allowAnyHosts = true
        sshCommand remote: remote, command: "curl -L -X GET 'http://${NEXUS_HOMEPAGE}/service/rest/v1/search/assets/download?sort=version&repository=clinistats-repo&group=dev&name=ribhus&maven.baseVersion=0.0.1-SNAPSHOT' --output /opt/ribhus_backend/ribhus-0.0.1-SNAPSHOT.jar"
        sshCommand remote: remote, command: "/opt/ribhus_backend/deploy.sh > /opt/ribhus_backend/deploy.log"
	    sshCommand remote: remote, command: "sleep 60"
	    sshCommand remote: remote, command: "cat /opt/ribhus_backend/deploy.log" 
	        }
            }
        }
    }
       post {
        always {
           script {
                        
            def recipients = emailextrecipients([ [$class: 'DevelopersRecipientProvider'],[$class: 'CulpritsRecipientProvider']])
            mail to: "${recipients}, devops_team@clinistat.onmicrosoft.com", 
            subject: "${JOB_NAME} :: Build# ${BUILD_NUMBER} :: Build Status# ${currentBuild.currentResult}", 
            body: "For more details ${env.BUILD_URL}"
            echo "${recipients}"


             }
        }
    } 
}
