node {

  // Get the source code
  //stage 'Checkout'

  // Get some code from a GitHub repository
  //checkout([$class: 'GitSCM', branches: [[name: '*/feature-automated-deployments']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CleanBeforeCheckout']], submoduleCfg: [], userRemoteConfigs: [[credentialsId: gitCredentialsId, url: gitRepo]]])
stage 'Checkout'

checkout scm

  // Mark the code build 'stage'....
  stage 'Build'
  
  
  // Build commands here (--no-progress can be added)
  //sh "bash build.sh"
  sh 'echo building...'
  sh 'ls -l'
  sh 'pwd'

  // Archive the output 
  stage 'Archive'

  // archive only the files we want, exclude anything not needed
  sh "cat include_files.txt | zip --symlinks -9 --quiet -r /tmp/${env.BUILD_TAG}.zip -@ -x@exclude_files.txt"

  // Upload to S3
  sh "aws s3 cp /tmp/${env.BUILD_TAG}.zip s3://${deploymentBucket}/${projectBaseName}/${env.BUILD_TAG}.zip"


  // Staging Environment
  stage 'Deploy Staging'
  
build job: 'deploy-codedeploy', parameters: [
  [$class: 'StringParameterValue', name: 'codedeployApp', value: "stage-${projectBaseName}"], 
  [$class: 'StringParameterValue', name: 'codedeployRegion', value: projectRegion], 
  [$class: 'StringParameterValue', name: 'deploymentPackage', value: "${projectBaseName}/${env.BUILD_TAG}.zip"], 
  [$class: 'StringParameterValue', name: 'codedeployConfig', value: 'CodeDeployDefault.OneAtATime'], 
  [$class: 'StringParameterValue', name: 'deploymentBucket', value: deploymentBucket],
  [$class: 'StringParameterValue', name: 'deploymentDescription', value: "Promotion from Jenkins: ${env.BUILD_URL}"]
]

  // production deployment
  stage 'Deploy Production'
  
  timeout(time:1, unit:'DAYS') {
    input message:'Approve deployment?'
  }
   
}
