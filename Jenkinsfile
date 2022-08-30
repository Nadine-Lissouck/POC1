import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=0204ac69-8342-4e69-8716-aecb5b5ccfa8',
        'AZURE_TENANT_ID=36da45f1-dd2c-4d1f-af13-5abe46b99921']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'PW-EnterpriseSPC-ResourceGroup'
      def webAppName = 'oacis-react-webapp-devops-test'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'ServicePrincipal', passwordVariable: '3Fh8Q~eTwyNCJ5wESpkgf3e2apfJSbNCNVfunb1m', usernameVariable: 'c8eda4d9-d837-4044-901d-0adeefae5681')]) {
       sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
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
