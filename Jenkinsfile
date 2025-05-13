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

    stage('Open Port 8080') {
      steps {
        sh '''
          set -ex
          az network nsg rule create \
            --resource-group rgUbuntuSoutheastAsia \
            --nsg-name nicDockerDemo01 \
            --name AllowHTTP \
            --protocol Tcp \
            --direction Inbound \
            --priority 1002 \
            --source-address-prefix '*' \
            --source-port-range '*' \
            --destination-address-prefix '*' \
            --destination-port-range 8080 \
            --access Allow || true
        '''
      }
    }

    stage('Get Public IP') {
      steps {
        sh '''
          set -ex
          . $ANSIBLE_ENV_PATH/bin/activate
          ansible-playbook playbooks/get-vm-ip.yaml -e "output_file=vm_ip.txt"
        '''
      }
    }

    stage('Verify Docker on VM') {
      steps {
        sh '''
          set -ex
          . $ANSIBLE_ENV_PATH/bin/activate
          ansible-playbook -i "$(cat vm_ip.txt)," \
            -u azureuser --private-key ${SSH_KEY} \
            playbooks/verify-docker.yaml
        '''
      }
    }
  }
}
