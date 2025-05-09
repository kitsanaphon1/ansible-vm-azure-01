pipeline {
    agent { label 'ansible-agent' }

    stages {
        stage('Run Ansible Playbook with venv') {
            steps {
                dir('/home/solo') {
                    sh '''
                        set -ex
                        source ~/ansible-azure-env/bin/activate
                        ansible-playbook create-linux-vm-01.yaml
                    '''
                }
            }
        }
    }
}
