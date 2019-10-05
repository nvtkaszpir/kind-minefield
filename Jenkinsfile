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
    lock resource: 'kind-cluster'

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

        kind version
        kind delete cluster || true
        kind create cluster --config kind-config.yaml --image "kindest/node:v1.13.10"

        '''
        // export KUBECONFIG env var used later by kind and kubectl
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
        kubectl get nodes
        kubectl get pods -A

        '''
      }
    }

    stage('k8s'){
      steps {
        sh '''

        kubectl apply -f provision/kubectl/

        '''
      }
    }

    stage('helm'){
      agent {
       docker {
        label "k8s"
        image 'alpine/helm:2.13.1'
        args '--network=host -v ${HOME}/.kube:/tmp/.kube:ro -e KUBECONFIG=/tmp/.kube/kind-config-kind -e HELM_HOME=/tmp --entrypoint="" '
      }
    }

    steps {
      sh '''

      helm init --service-account tiller --wait
      helm version

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
