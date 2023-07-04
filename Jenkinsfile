import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=5e209e08-2b5b-4f6b-a413-bc6333520772',
        'AZURE_TENANT_ID=dcfbcaec-dc40-43b7-a053-b235fbdd76f8']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'azurejenkins'
      def webAppName = 'azurejenkinsworkshop'
      // login Azure
      withCredentials([usernamePassword(credentialsId: '71ebbb7b-ed8a-43d1-8ab9-3b2c5e7aa7d6', passwordVariable: 'EaL8Q~8rNQzBPjsmxHnSYSkZCagnsUbtjkMQbcMS', usernameVariable: '71ebbb7b-ed8a-43d1-8ab9-3b2c5e7aa7d6')]) {
       sh '''
          az login --service-principal -u 71ebbb7b-ed8a-43d1-8ab9-3b2c5e7aa7d6 -p EaL8Q~8rNQzBPjsmxHnSYSkZCagnsUbtjkMQbcMS -t dcfbcaec-dc40-43b7-a053-b235fbdd76f8
          az account set -s 5e209e08-2b5b-4f6b-a413-bc6333520772
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
