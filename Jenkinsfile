pipeline {
  agent {
    label "jenkins-jx-base"
  }
  environment {
    DEPLOY_NAMESPACE = "jx"
  }
  stages {
    stage('Download secrets yaml') {
      steps {
        container('jx-base') {
          dir('env') {
            withCredentials([file(credentialsId: 'gitops-secrets', variable: 'FILE')]) {
              sh "mv ${FILE} ./secrets.yaml"
            }
          }
        }
      }
    }
    stage('Setup kube env') {
      steps {
        container('jx-base') {
          withCredentials([file(credentialsId: 'kubeconfig', variable: 'FILE')]) {
            sh label: "Kube Config", script: "mkdir -p ~/.kube && mv ${FILE} ~/.kube/config"
            sh label: "Kube use-context",
                    script: "kubectl config use-context gke_jenkins-x-235314_us-central1-a_jenkinsx-ng"
          }
        }
      }
    }
    stage('Validate Environment') {
      steps {
        container('jx-base') {
          dir('env') {
            sh 'jx ctx -b'
            sh 'jx step helm build'
          }
        }
      }
    }
    stage('Update Environment') {
      when {
        branch 'master'
      }
      steps {
        container('jx-base') {
          dir('env') {
            sh 'jx ctx -b'
            sh 'jx step env apply'
          }
        }
      }
    }
  }
}
