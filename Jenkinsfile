import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID= 18627877-22bf-4741-88c8-a951f6fa36de',
        'AZURE_TENANT_ID= dab5b5db-6cee-44a7-892c-30a42811f9d1']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'myTFResourceGroup'
      def webAppName = 'Test1234Jam'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'password', usernameVariable: 'appId')]) {
       sh '''
          az login --service-principal -u 16972635-7cc2-4a9b-8047-4ae54e67f2a2 -p CEnS-ip.eUAfLOMFCtiOj84B0NMboA.pmp --tenant dab5b5db-6cee-44a7-892c-30a42811f9d1
          az account set -s $AZURE_SUBSCRIPTION_ID
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
