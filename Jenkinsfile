properties([
	parameters([
        string(defaultValue: "master", description: 'Which Git Branch to clone?', name: 'GIT_BRANCH')
	])
])

try {

  stage('Checkout'){
    node('master'){
        checkout([$class: 'GitSCM', branches: [[name: '*/$GIT_BRANCH']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/MKrupA5/examplapp.git']]])
    }
  }

  stage('create container image') {
      container('docker') {
        sh """
          docker build -t exampleapp .
        """
      }
  }

  stage('Deploy on Dev') {
  	node('master'){
        sh ''' sed -i 's/VERSION/'"${BUILD_NUMBER}"'/g' deploy/flask.yml '''
  	    sh "kubectl apply -f deploy/*"
	DEPLOYMENT = sh (
	        script: 'grep metadata -A3 deploy/flask.yml|grep name|awk "{print $2}"| head -1 | tr -s " " |cut -d " " -f 3',
	        returnStdout: true
	).trim()
	echo "Creating the deployment..."
	sleep 60
	DESIRED= sh (
          	script: "kubectl get deployment/$DEPLOYMENT | awk '{print \$2}' | grep -v READY | cut -b 1 ",
          	returnStdout: true
        ).trim()
        CURRENT= sh (
          	script: "kubectl get deployment/$DEPLOYMENT | awk '{print \$2}' | grep -v READY | cut -b 1 ",
          	returnStdout: true
        ).trim()
        if (DESIRED.equals(CURRENT)) {
          	currentBuild.result = "SUCCESS"
          	return
        } else {
          	error("Deployment Unsuccessful.")
          	currentBuild.result = "FAILURE"
          	return
        }
      	}
    }
}
catch (err){
    currentBuild.result = "FAILURE"
    throw err
}
