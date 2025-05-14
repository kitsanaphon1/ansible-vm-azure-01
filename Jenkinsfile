/**
 * 🔧 Jenkins Pipeline สำหรับจัดการ VM บน Azure ด้วย Ansible
 *
 * ✅ วิธีใช้งาน:
 * 1. DESTROY_MODE = false → สร้าง VM + ติดตั้ง Docker + ตรวจสอบ
 * 2. DESTROY_MODE = true → ลบ VM เดิมทิ้ง
 */

pipeline {
  agent { label 'ansible-agent' }

  parameters {
    booleanParam(name: 'DESTROY_MODE', defaultValue: false, description: 'ติ๊กเพื่อสั่งลบ VM แทนการสร้าง')
  }

  environment {
    SSH_KEY = "~/.ssh/id_rsa"
    ANSIBLE_ENV_PATH = "/var/lib/jenkins/ansible-azure-env"
  }

  stages {
    stage('Clean Workspace') {
      steps {
        echo "🧹 ล้าง workspace..."
        cleanWs()
      }
    }

    stage('Checkout Source') {
      steps {
        echo "📥 ดึง source code จาก Git..."
        checkout scm
      }
    }

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
                export ANSIBLE_HOST_KEY_CHECKING=False
                . $ANSIBLE_ENV_PATH/bin/activate
                ansible-playbook playbooks/destroy-linux-vm.yaml
              '''
            } else {
              echo "🚀 DESTROY_MODE = false → สร้าง VM และตรวจสอบ Docker"

              // ▶ 1. สร้าง VM
              sh '''
                set -ex
                export ANSIBLE_HOST_KEY_CHECKING=False
                . $ANSIBLE_ENV_PATH/bin/activate
                ansible-playbook playbooks/create-linux-vm-02.yaml
              '''

              // ▶ 2. ดึง Public IP และบันทึกลง workspace
              sh '''
                set -ex
                export ANSIBLE_HOST_KEY_CHECKING=False
                . $ANSIBLE_ENV_PATH/bin/activate
                ansible-playbook playbooks/get-vm-ip.yaml -e "output_file=vm_ip.txt"
                echo "📄 ตรวจ vm_ip.txt:"
                cat vm_ip.txt | hexdump -C
              '''

              // ▶ 3. อ่าน IP แบบปลอดภัยและ SSH ไปเช็ก Docker
              sh '''
                set -ex
                export ANSIBLE_HOST_KEY_CHECKING=False

                if [ ! -s vm_ip.txt ]; then
                  echo "❌ vm_ip.txt ว่างหรือไม่ถูกเขียน!"
                  exit 1
                fi

                IP=$(cat vm_ip.txt | tr -d '\\r\\n')
                if [ -z "$IP" ]; then
                  echo "❌ ไม่พบ IP ในไฟล์ vm_ip.txt"
                  exit 1
                fi

                echo "🌐 Connecting to VM: $IP"
                . $ANSIBLE_ENV_PATH/bin/activate
                ansible-playbook -i "$IP," \
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
