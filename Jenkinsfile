/**
 * 🔧 Jenkins Pipeline สำหรับจัดการ VM บน Azure ด้วย Ansible
 *
 * ✅ วิธีใช้งาน:
 * 1. รัน Pipeline ปกติ (DESTROY_MODE = false) → จะสร้าง VM + ติดตั้ง Docker + ตรวจสอบ
 * 2. รัน Pipeline โดยตั้ง DESTROY_MODE = true → จะลบ VM เดิมทิ้ง
 *
 * 🚀 เหมาะสำหรับ Dev/Test ที่ต้องการความยืดหยุ่นในการสร้าง-ลบ VM บ่อย ๆ
 */

pipeline {
  agent { label 'ansible-agent' }

  parameters {
    booleanParam(name: 'DESTROY_MODE', defaultValue: true, description: 'เช็คเพื่อสั่งลบ VM แทนการสร้าง')
  }

  environment {
    SSH_KEY = "~/.ssh/id_rsa"
    ANSIBLE_ENV_PATH = "/var/lib/jenkins/ansible-azure-env"
    ANSIBLE_HOST_KEY_CHECKING = "False"
  }

  stages {
    stage('Provision or Destroy') {
      steps {
        withCredentials([
          string(credentialsId: 'AZURE_CLIENT_ID', variable: 'AZURE_CLIENT_ID'),
          string(credentialsId: 'AZURE_SECRET', variable: 'AZURE_SECRET'),
          string(credentialsId: 'AZURE_TENANT', variable: 'AZURE_TENANT'),
          string(credentialsId: 'AZURE_SUBSCRIPTION_ID', variable: 'AZURE_SUBSCRIPTION_ID')
        ]) {
          script {
            if (params.DESTROY_MODE) {
              echo "🔥 DESTROY_MODE = true → ลบ VM"
              sh '''
                set -ex
                . $ANSIBLE_ENV_PATH/bin/activate
                ansible-playbook playbooks/destroy-linux-vm.yaml
              '''
            } else {
              echo "🚀 DESTROY_MODE = false → สร้าง VM และตรวจสอบ Docker"
              sh '''
                set -ex
                . $ANSIBLE_ENV_PATH/bin/activate
                ansible-playbook playbooks/create-linux-vm-02.yaml
              '''

              sh '''
                set -ex
                . $ANSIBLE_ENV_PATH/bin/activate
                ansible-playbook playbooks/get-vm-ip.yaml -e "output_file=vm_ip.txt"
              '''

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
    }
  }
}
