stage 'Compile'
node('linux1') {
    checkout scm
    // use for non multibranch: git 'https://github.com/amuniz/maven-helloworld.git'
    System.out.println "1 CURRENT BUILD RESULT: " + currentBuild.result;
    try {
      def mvnHome = tool 'maven-3'
      System.out.println "2 CURRENT BUILD RESULT: " + currentBuild.result;
      sh "${mvnHome}/bin/mvn clean install -DskipTests"
      System.out.println "3 CURRENT BUILD RESULT: " + currentBuild.result;
      stash 'working-copy'
      System.out.println "4 CURRENT BUILD RESULT: " + currentBuild.result;
    } catch(e) {
      currentBuild.result = 'FAILURE'
      throw e
    } finally {
        System.out.println "5 CURRENT BUILD RESULT: " + currentBuild.result;
        echo "notify_fixed_or_failed..."
        notify_fixed_or_failed();
        System.out.println "6 CURRENT BUILD RESULT: " + currentBuild.result;
    }

    def changeLogSets = currentBuild.rawBuild.changeSets
    echo "ChangeLogSets: " + changeLogSets
    for (int i = 0; i < changeLogSets.size(); i++) {
       def entries = changeLogSets[i].items
       for (int j = 0; j < entries.length; j++) {
           def entry = entries[j]
           echo "${entry.commitId} by ${entry.author} on ${new Date(entry.timestamp)}: ${entry.msg}"
           def files = new ArrayList(entry.affectedFiles)
           for (int k = 0; k < files.size(); k++) {
               def file = files[k]
               echo "  ${file.editType.name} ${file.path}"
           }
       }
    }
}

stage 'Test'
parallel one: {
    node('linux1') {
        unstash 'working-copy'
        def mvnHome = tool 'maven-3'
        sh "${mvnHome}/bin/mvn test -Diterations=10"
    }
}, two: {
    node('linux1') {
        unstash 'working-copy'
        def mvnHome = tool 'maven-3'
        sh "${mvnHome}/bin/mvn test -Diterations=5"
    }
}, failFast: true


def notify_fixed_or_failed(String recipients=null) {
  System.out.println "CURRENT BUILD RESULT: " + currentBuild.result;
  if (is_fixed()) {
    echo "sending FIXED email"
    send_email('FIXED', recipients)
  } else if (get_status() == 'FAILURE') {
    echo "sending FAILURE email"
    send_email('FAILURE', recipients)
  }
  System.out.println "CURRENT BUILD RESULT: " + currentBuild.result;
}

def send_email(String status=null, String recipients=null) {

  status = status ?: get_status()
  def subject = "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"

  def details = """
  <p>Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
  <p>Check console output at "<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>"</p>
  """

  System.out.println "[send_email] CURRENT BUILD RESULT: " + currentBuild.result;
  if (recipients) {
    System.out.println "Email to recipients"
    emailext(subject: subject, body: details, to: recipients)
  } else {
    recipientsProvider = [
      [$class: 'CulpritsRecipientProvider'],
      [$class: 'DevelopersRecipientProvider'],
      [$class: 'RequesterRecipientProvider']
    ]
    System.out.println "Email to recipientsProvider"
    emailext(subject: subject, body: details, recipientProviders: recipientsProvider)
  }
  System.out.println "[send_email] CURRENT BUILD RESULT: " + currentBuild.result;
}

def is_fixed() {
    def status = get_status()
    echo status
    return (status == 'SUCCESS' && !hudson.model.Result.SUCCESS.equals(currentBuild.rawBuild.getPreviousBuild()?.getResult()))
}
def get_status() {
    def status = currentBuild.result ?: 'SUCCESS'
    currentBuild.result = status
    return status
}
