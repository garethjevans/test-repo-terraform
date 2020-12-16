def terraformTemplate = """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: terraform
    image: hashicorp/terraform:0.13.5
    command:
    - cat
    tty: true
"""

pipeline {
  agent {
    kubernetes {
      yaml terraformTemplate
    }
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    timeout(time: 30, unit: 'MINUTES')
    disableConcurrentBuilds()
  }

  stages {
    stage('Dummy') {
      steps {
        container('terraform') {
          sh 'terraform init'
          sh 'terraform validate'
          sh 'curl --silent --location --show-error --output tfsec https://github.com/tfsec/tfsec/releases/download/v0.36.9/tfsec-linux-amd64'
          sh 'chmod a+x tfsec'
          sh './tfsec'
          script {
            plan = sh(returnStdout: true, script: 'terraform plan -no-color')
            pullRequest.comment('```\n' + plan)
          }
        }
      }
    }
  }
}
