pipeline {
    agent { label 'ansible-agent' }

    environment {
        SSH_KEY = "~/.ssh/id_rsa"
        ANSIBLE_HOST_KEY_CHECKING = "False"  // ✅ ปิด host key checking
    }

    stages {
        stage('Create VM with Docker') {
            steps {
                sh '''
                    set -ex
                    . ~/ansible-azure-env/bin/activate
                    ansible-playbook playbooks/create-linux-vm-02.yaml
                '''
            }
        }

        stage('Open Port 80') {
            steps {
                sh '''
                    set -ex
                    . ~/ansible-azure-env/bin/activate
                    ansible-playbook playbooks/open-port-80.yaml
                '''
            }
        }

        stage('Get Public IP') {
            steps {
                script {
                    def ip_output = sh(
                        script: '''
                        . ~/ansible-azure-env/bin/activate
                        ansible-playbook playbooks/get-vm-ip.yaml -e "output_file=vm_ip.txt"
                        ''',
                        returnStdout: true
                    )
                    echo ip_output
                }
            }
        }

        stage('Verify Docker on VM') {
            steps {
                sh '''
                    set -ex
                    . ~/ansible-azure-env/bin/activate
                    ansible-playbook -i "$(cat vm_ip.txt)," \
                        -u azureuser --private-key ${SSH_KEY} \
                        playbooks/verify-docker.yaml
                '''
            }
        }
    }
}
