pipeline {
    agent { label 'ansible-agent' }

    stages {
        stage('Run Ansible Playbook with venv') {
            steps {
                dir('/home/solo') { // 🔁 เปลี่ยนเป็น path ที่ไฟล์ .yaml อยู่จริง
                    sh '''
                        source ~/ansible-azure-env/bin/activate
                        ansible-playbook create-linux-vm-01.yaml
                    '''
                }
            }
        }
    }
}
