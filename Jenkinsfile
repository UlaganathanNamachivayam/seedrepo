pipeline {
  	
  environment {
    registry = "coolbud/playground"
    registryCredential = 'playground_docker'
    dockerImage = ''
    user_email = "ulags.n@gmail.com"
  }
  
  agent any
  
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
  }
 post {
		success {
			emailext (recipientProviders: [[$class: 'RequesterRecipientProvider'], [$class: 'DevelopersRecipientProvider']], to: "${env.user_email}", subject:"BUILD & DEPLOYMENT SUCCESS: ${currentBuild.fullDisplayName}", body: "Build & Deployment Successful! New Build Deployed with following changes: ${currentBuild.absoluteUrl}changes. Reports Attached. Please review the reports and take necessary actions.")
			cleanWs()
		}

		failure {
			emailext (recipientProviders: [[$class: 'CulpritsRecipientProvider'], [$class: 'RequesterRecipientProvider']], to: "${env.user_email}", subject:"BUILD & DEPLOYMENT FAILURE: ${currentBuild.fullDisplayName}", body: "Build & Deployment Failed! Your commits is suspected to have caused the build failure. Please go to ${BUILD_URL} for details and resolve the build failure at the earliest.", attachLog: false, compressLog: false)
			cleanWs()
		}

		aborted {
			emailext (recipientProviders: [[$class: 'RequesterRecipientProvider'], [$class: 'DevelopersRecipientProvider']], to: "${env.user_email}", subject:"BUILD & DEPLOYMENT ABORTED: ${currentBuild.fullDisplayName}", body: "Build & Deployment Aborted! Please go to ${BUILD_URL} and verify the build.", attachLog: false, compressLog: false)
			cleanWs()
		}
	}

}

