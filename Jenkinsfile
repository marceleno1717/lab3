pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    stages {
        stage('Ansible Version Check') {
            steps {
                sh 'ansible --version'
                sh 'ansible-playbook --version'
            }
        }

        stage('Ping Target Machine') {
            steps {
                sh 'ansible -i inventory.ini all -m ping'
            }
        }

        stage('Execute Playbook') {
            steps {
                sh 'ansible-playbook -i inventory.ini playbook.yml'
            }
        }
    }

    post {
        success {
            echo 'Ansible playbook completed successfully!'
        }
        failure {
            echo 'Ansible playbook execution failed!'
        }
    }
}
