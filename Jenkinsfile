import hudson.plugins.git.util.BuildData

@NonCPS
def getLastRevision() {
  def lastBuild = rawBuild.getLastSuccessfulBuild();
  def commitId = lastBuild?.getAction(BuildData)?.lastBuiltRevision.sha1;
}

pipeline {
  agent any

  stages {
    stage('Update changelog') {
      when {
        anyOf {
          branch 'master'
          expression { BRANCH_NAME ==~ /^\d+(.\d+)*$/ }
        }
      }

      steps {
        sh "docker pull jsternberg/changelog"
        withDockerContainer(image: "jsternberg/changelog") {
          withCredentials(
            [[$class: "UsernamePasswordMultiBinding",
              credentialsId: "hercules-username-password",
              usernameVariable: "GITHUB_USER",
              passwordVariable: "GITHUB_TOKEN"]]) {
            script {
              def commitId = getLastRevision();
              if (commitId) {
                sh "git changelog ${commitId}"
              } else {
                sh "git changelog"
              }
            }
          }
        }

        sshagent(credentials: ['jenkins-hercules-ssh']) {
          sh """
          set -e
          if ! git diff --quiet; then
            git -c user.name='Hercules Mango Churchill' -c user.email='hercules@influxdata.com' commit -am 'Update changelog'
            git -c remote.origin.pushurl=git@github.com:influxdata/changelog-test.git push origin HEAD:${BRANCH_NAME}
          fi
          """
        }
      }
    }
  }
}
