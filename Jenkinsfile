import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=f3c5cb8c-25f2-44cd-ad2d-a9b2109b9f5a',
        'AZURE_TENANT_ID=db53b663-72da-481d-ac43-e5a95210ba09']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'workshop'
      def webAppName = 'new-application-w'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'new-work', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
       sh '''
          az login --service-principal -u d20e5843-0b7a-4f8b-a913-18d2f6ec5f45 -p njD8Q~QXkQkrFhPON2XDyKlbBLkw2tX-GU~vncIU -t db53b663-72da-481d-ac43-e5a95210ba09
          az account set -s f3c5cb8c-25f2-44cd-ad2d-a9b2109b9f5a
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
