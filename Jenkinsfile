pipeline {
    agent { label 'ansible-agent' }

    stages {
        stage('Run Ansible Playbook with venv') {
            steps {
                sh '''
                    set -ex
                    . ~/ansible-azure-env/bin/activate
                    ansible-playbook /home/solo/create-linux-vm-01.yaml
                '''
            }
        }
    }
}
