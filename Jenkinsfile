#!/usr/bin/env groovy

def job_desc = """
Jenkins multibranch job for kind-minefiled
<br/>
<a href='https://github.com/nvtkaszpir/kind-minefield'>github</a>
"""

pipeline {

  agent { label "k8s"}

  options {
    timeout(time: 1, unit: 'HOURS')
    disableConcurrentBuilds()
    ansiColor('xterm')
    timestamps() // breaks ansiColor plugin
  }

  // notice that parameters are dynamically updated in first stage
  // parameters {
  // }

  stages {

    stage('Reconfig job'){
      steps {
        script{
          def job = Jenkins.instance.getItemByFullName(job_name)
          // change description
          def pr_desc = ""
          if ( env.CHANGE_ID != null) {
            pr_desc += "<br/>"
            pr_desc += "<br/>"
            pr_desc += "<br/>"
            pr_desc += "<h2>Change URL: <a href='${env.CHANGE_URL}'>${env.CHANGE_TITLE}</a></h2><br/>"
          }
          job.setDescription(job_desc + pr_desc)
          job.save()
        }
      }
    }

    stage('KinD up'){
      steps {
        sh '''
        kind create cluster
        '''
        script {
          env.KUBECONFIG = sh(
            returnStdout: true,
            script: 'kind get kubeconfig-path --name="kind"'
            ).trim()
        }
      }
    }

    stage('KinD info'){
      steps {
        sh '''
        export | sort
        kubectl cluster-info
        '''
      }
    }


  }


  post {
    always {
      sh '''
      kind delete cluster
      '''

      archiveArtifacts allowEmptyArchive: true,  artifacts: '**/reports/*', excludes: '**/*.gitkeep', fingerprint: true
      deleteDir()
    }
  }
}
