pipeline {
  	
  environment {
    registry = "coolbud/playground"
    registryCredential = 'playground_docker'
    dockerImage = ''
    useremail = ''
  }
  
  agent any
	
	// parameters { string(defaultValue: "${env.useremail}", description: 'email for notifications', name: 'useremail') }
	
  //parameters {
 //	  string(name: 'useremail', defaultValue: "${env.useremail}", description: 'Candidate email id')
 // }
	
  stages {
	  
    stage('Checkout') {
      steps {
        git "${git_url}"
      }
    }
    
    stage('BuildingImage') {
      steps{
        script {
           dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    
    stage('PushingImage') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
      stage('DeployingImage') {
        steps{
          script {
            dockerImage.pull()
            dockerImage.run('-p 80:80') // {c -> sh 'sleep 2m' }
            // sh "ssh -tt ciuser@localhost && sudo docker run -d -p 80:80 registry:$BUILD_NUMBER"
          }
      }  
    }
	  
	  stage('CommiterEmail'){
       	    steps {
	       script { 
		       	useremail = sh (
		        script: 'git --no-pager show -s --format=\'%ae\'',
	       	        returnStdout: true
			).trim()       
	        // sh "useremail = $(git --no-pager show -s --format='%ae' $GIT_COMMIT)"
	       }
	    }
	  }
	  
	  stage('BuildResult') {
	      steps {
		 script { 
			emailext attachLog: true, body:
			   """<p>EXECUTED: Job <b>\'${env.JOB_NAME}:${env.BUILD_NUMBER})\'
			   </b></p><p>View console output at "<a href="${env.BUILD_URL}"> 
			   ${env.JOB_NAME}:${env.BUILD_NUMBER}</a>"</p> 
			     <p><i>(Build log is attached.)</i></p>""", 
			    compressLog: true,
			    recipientProviders: [[$class: 'DevelopersRecipientProvider'], 
			     [$class: 'RequesterRecipientProvider']],
			    subject: "Status: ${currentBuild.result?:'SUCCESS'} - Job \'${env.JOB_NAME}:${env.BUILD_NUMBER}\'",
		            to: "${env.useremail}"
		      
	    } 
         }  
      }
  }
 post {
		success {
			emailext (recipientProviders: [[$class: 'RequesterRecipientProvider'], [$class: 'DevelopersRecipientProvider']], to: "${env.useremail}", subject:"BUILD & DEPLOYMENT SUCCESS: ${currentBuild.fullDisplayName}", body: "Build & Deployment Successful! New Build Deployed with following changes: ${currentBuild.absoluteUrl}changes. Reports Attached. Please review the reports and take necessary actions.")
			cleanWs()
		}

		failure {
			emailext (recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'RequesterRecipientProvider']], to: "${env.useremail}", subject:"BUILD & DEPLOYMENT FAILURE: ${currentBuild.fullDisplayName}", body: "Build & Deployment Failed! Your commits is suspected to have caused the build failure. Please go to ${BUILD_URL} for details and resolve the build failure at the earliest.", attachLog: false, compressLog: false)
			cleanWs()
		}

		aborted {
			emailext (recipientProviders: [[$class: 'RequesterRecipientProvider'], [$class: 'DevelopersRecipientProvider']], to: "${env.useremail}", subject:"BUILD & DEPLOYMENT ABORTED: ${currentBuild.fullDisplayName}", body: "Build & Deployment Aborted! Please go to ${BUILD_URL} and verify the build.", attachLog: false, compressLog: false)
			cleanWs()
		}
	}

}

