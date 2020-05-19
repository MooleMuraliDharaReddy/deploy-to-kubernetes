pipeline {
agent any
triggers {
	//Execute  every five minute
cron('H/5 * * * *')
}

options{
// only keeps the last 10 builds of this pipelone
buildDiscarder(logRotator(numToKeepStr: '10'))
 // discardConcurrentBuilds()
//disableConcurrentBuilds

// adds timestamps to build logs
timestamps()
}
tools
{
maven 'maven'
}
stages {
	


  stage('Build Docker Image'){
	  steps {
        sh 'docker build -t mmreddy424/spring-boot-mongo .'
  }
    }

    stage('Push Docker Image'){
	    steps {
        withCredentials([string(credentialsId: 'DOKCER_HUB_PASSWORD', variable: 'Docker_Password')]) {
          sh "docker login -u mmreddy424 -p ${Docker_Password}"
        }
        sh 'docker push mmreddy424/spring-boot-mongo'
	    }
     }

     stage("Deploy To Kuberates Cluster"){
	     steps {
       kubernetesDeploy(
         configs: 'springBootMongo.yml',
         kubeconfigId: 'KUBERNATES_CONFIG',
         enableConfigSubstitution: true
        )
	     }
	 }
	 


}
}
