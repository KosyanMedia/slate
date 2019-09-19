@Library('jenkins-pipeline-shared') _

def notifier = new org.gradiant.jenkins.slack.SlackNotifier()
def job_name = env.JOB_NAME.split('/')[0]
def project_name = "slate"
def repo_prefix = "github.com/KosyanMedia"

podTemplate(
  label: "${job_name}",
  imagePullSecrets: ['dockerhub'],
  containers: [
     containerTemplate(name: 'ruby',
      image: "library/ruby:2.5-alpine",
      ttyEnabled: true,
      command: 'cat'
    ),
  ]) {

  node("${job_name}") {
    try {
      env.NOTIFY_SUCCESS = 'true'
      env.SLACK_CHANNEL = "#build"

      stage('Checkout repo') {
        notifier.notifyStart()
        container('ruby') {
          checkout(scm)
        }
      }
      stage("Prepare and build") {
        container('ruby') {
          sh "apk add --update nodejs g++ make openssh bash"
          sh "bundle install"
          sh "./deploy.sh --source-only"
        }
      }
      stage('Install ssh keys') {
        container('ruby') {
          withCredentials([file(credentialsId: 'hackaton2019-ssh-key', variable: 'SecretFile')]) {
            sh """
              mkdir /root/.ssh
              cp $SecretFile /root/.ssh/id_rsa
              chmod 0700 /root/.ssh
              chmod 0600 /root/.ssh/id_rsa
            """
          }
        }
      }
      stage("Deploy") {
        container('ruby') {
          withCredentials([file(credentialsId: 'hackaton2019-ssh-key', variable: 'SecretFile')]) {
            sh "scp -r -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -P2200 build/* hackaton2019@hackaton2019.aviasales.ru:~/data/www/hackaton2019.aviasales.ru/"
          }
        }
      }
    }
    catch (err) {
      currentBuild.result = 'FAILURE'
      notifier.notifyError(err)
      throw err
    }
    finally {
      notifier.notifyResult()
    }
  }
}
