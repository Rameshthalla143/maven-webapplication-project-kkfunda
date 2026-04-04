node {

   echo "git branch name: ${env.BRANCH_NAME}"
   echo "build number is: ${env.BUILD_NUMBER}"
   echo "node name is: ${env.NODE_NAME}"

   def mavenHome = tool name: "maven3.9.9"

   try {

      stage('git checkout') {
        notifyBuild('STARTED')
        git branch: 'development', url: 'https://github.com/Rameshthalla143/maven-webapplication-project-kkfunda.git'
      } 

      stage('COMPILE') {
        sh "${mavenHome}/bin/mvn clean compile"
      }

      stage('Build') {
        sh "${mavenHome}/bin/mvn clean package"
      }

      stage('SQ Report') {
        sh "${mavenHome}/bin/mvn sonar:sonar"
      }

      stage('Upload Artifact') {
        sh "${mavenHome}/bin/mvn clean deploy"
      }

      stage('Deploy to Tomcat') {
        sh """
        curl -u admin:kkfunda \
        --upload-file target/maven-web-application.war \
        "http://13.200.251.28:9090/manager/text/deploy?path=/maven-web-application&update=true"
        """
      }

   } catch (e) {

      currentBuild.result = "FAILED"
      throw e   // ✅ important (so Jenkins marks build failed properly)

   } finally {

      notifyBuild(currentBuild.result)

   }
}


// ✅ FIXED FUNCTION
def notifyBuild(String buildStatus = 'STARTED') {

  buildStatus = buildStatus ?: 'SUCCESS'

  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // ✅ FIX: use 'def color' (avoid warning)
  def color = 'RED'

  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // ✅ SAFE SLACK CALL
  try {
    slackSend(color: colorCode, message: summary, channel: '#jio-devteam')
    slackSend(color: colorCode, message: summary, channel: '#jio-devopsteam')
  } catch (err) {
    echo "Slack notification failed: ${err}"
  }
}
