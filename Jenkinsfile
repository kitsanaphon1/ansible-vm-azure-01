pipeline {
  agent { label 'ansible-agent' }

  environment {
    SSH_KEY = "~/.ssh/id_rsa"
    ANSIBLE_ENV_PATH = "/var/lib/jenkins/ansible-azure-env"
    ANSIBLE_HOST_KEY_CHECKING = "False"
  }

  stages {
    stage('Create VM with Docker') {
      steps {
        withCredentials([
          string(credentialsId: 'AZURE_CLIENT_ID', variable: 'AZURE_CLIENT_ID'),
          string(credentialsId: 'AZURE_SECRET', variable: 'AZURE_SECRET'),
          string(credentialsId: 'AZURE_TENANT', variable: 'AZURE_TENANT'),
          string(credentialsId: 'AZURE_SUBSCRIPTION_ID', variable: 'AZURE_SUBSCRIPTION_ID')
        ]) {
          sh '''
            set -ex
            . $ANSIBLE_ENV_PATH/bin/activate
            ansible-playbook playbooks/create-linux-vm-02.yaml
          '''
        }
      }
    }

    // ...สามารถเพิ่ม stage อื่น ๆ ตามหลังได้เลย เช่น Open Port, Get IP, Verify Docker
  }
}
