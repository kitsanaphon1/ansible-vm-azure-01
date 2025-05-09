pipeline {
    agent { label 'ansible-agent' }

    stages {
        stage('Run Ansible Playbook') {
            steps {
                dir('/home/solo') { // 🔁 เปลี่ยนเป็น path ที่ไฟล์ .yaml อยู่จริง
                    sh 'ansible-playbook create-linux-vm.yaml'
                }
            }
        }
    }
}
