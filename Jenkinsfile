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
	stage('setup')
{
steps
  {
/* set all variariables for job *? */
/*in order to put all the variables craeted in this script block use the following syntax: sh([script: "echo \"Value:$GIT_COMMIT}\]) */
script
{
BRANCH_NAME=sh(script:"echo $GIT_BRANCH|sed -e 's|origin/||g'",returnStdout:true).trim()
  if("$BRANCH_NAME}"=='master')
  {
    echo"BRANCH_NAME:${BRANCH_NAME}"
    BUILD_VERSION='2019.99.0'
    echo "BUILD_VERSION=${BUILD_VERSION}"
  }
  else
  {
    BUILD_VERSION=sh(script:"echo $GIT_BRANCH |sed -e 's|origin/release.||g'",returnStdout:true).trim()
    echo "BUILD_VERSION=${BUILD_VERSION}"
  }
  /*component id need for depoy job */
  UDEPLOY_COMPONENT_ID='component id'
  GIT_COMMIT= sh([script: "git rev-parse HEAD",returnStdout:true]).trim()
  NEXUS_VERSION= sh([script: "git rev-list ${GIT_COMMIT} --count",returnStdout:true]).trim()
  GIT_SIMPLE= sh([script: "git rev-list ${GIT_COMMIT} |head -n 1 |cut -c 1-5",returnStdout:true]).trim()
  //get latest successful build name
  build =currentBuild
  while (build != null && build.result != 'SUCCESS')
	{
  build=build.previousBuild
}
    if(build == null)
    {
      //this means there has not been a success fill build yet
      PREVIOUS_BUILD_VERSION='none'
    }
    else
    {
      PREVIOUS_BUILD_VERSION=build.displayName
    }
    echo "LAST SUCCESS BUILD NAME: ${PREVIOUS_BUILD_VERSION}"
    //srt jenkisn build name to be same as artifact version that will be published
    currentBuild.displayName="${BUILD_VERSION}.${NEXUS_VERSION}-${GIT_SIMPLE}"
  }
}
}

  stage('Build Docker Image'){
	  steps
  {
        sh 'docker build -t mmreddy424/spring-boot-mongo .'
  }
    }

    stage('Push Docker Image'){
        withCredentials([string(credentialsId: 'DOKCER_HUB_PASSWORD', variable: 'Docker_Password')]) {
          sh "docker login -u mmreddy424 -p ${Docker_Password}"
        }
        sh 'docker push mmreddy424/spring-boot-mongo'
     }

     stage("Deploy To Kuberates Cluster"){
       kubernetesDeploy(
         configs: 'springBootMongo.yml',
         kubeconfigId: 'KUBERNATES_CONFIG',
         enableConfigSubstitution: true
        )
     
	 }
	 


}
}
