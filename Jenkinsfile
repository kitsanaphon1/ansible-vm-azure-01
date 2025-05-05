pipeline {
  agent any

  parameters {
    string(name: 'BRANCH_NAME', defaultValue: 'dev', description: 'Git branch to deploy')
  }

  stages {
    stage('Clone Repository') {
      steps {
        git branch: "${params.BRANCH_NAME}",
            url: 'https://github.com/kitsanaphon1/ansible-vm-azure-01.git'
      }
    }

    stage('Run Ansible Playbook') {
      steps {
        sh '''
          ssh -o StrictHostKeyChecking=no -i ~/.ssh/ansible_azure boo@4.194.250.173 \
            "ansible-playbook ~/ansible/playbooks/test-playbook.yml"
        '''
      }
    }
  }
}
