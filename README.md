# NTI Jenkins Lab 3: Ansible SSH Target Machine Setup

This lab demonstrates how `nti_jen` executes Ansible playbooks against a machine container named **`servera`** using **SSH** with root credentials.

---

## 1. Architecture

```
+--------------------------------------------------------+
|                      Docker Host                       |
|                                                        |
|  +--------------------+        +--------------------+  |
|  |     nti_jen        |  SSH   |      servera       |  |
|  | (Jenkins + Ansible)|------->| (SSH Server : 22)  |  |
|  |                    | (123)  | (Ubuntu 24.04 Web) |  |
|  +--------------------+        +--------------------+  |
|            \                      /                    |
|             \--- [lab-net bridge] ---/                 |
+--------------------------------------------------------+
```

- **Network**: `lab-net`
- **Target Container**: `servera`
- **User**: `root`
- **Password**: `123`

---

## 2. Target Machine Container Details

The container `servera` was built with `openssh-server`, `python3`, root password `123`, and is attached to `lab-net`:

```bash
docker run -d --name servera --network lab-net -p 2222:22 servera-image
```

---

## 3. Inventory Configuration ([`inventory.ini`](file:///home/marceleno/nti/jenkins_labs/lab3/inventory.ini))

```ini
[web]
servera ansible_host=servera ansible_user=root ansible_ssh_pass=123

[web:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null'
```

---

## 4. Playbook ([`playbook.yml`](file:///home/marceleno/nti/jenkins_labs/lab3/playbook.yml))

The playbook performs:
1. Connectivity check & fact gathering (`ansible.builtin.setup`).
2. Node info display.
3. Installs and updates `nginx`.
4. Deploys custom [`index.html`](file:///home/marceleno/nti/jenkins_labs/lab3/index.html) to `/var/www/html/index.html`.
5. Starts the `nginx` service.

---

## 5. Jenkins Pipeline ([`Jenkinsfile`](file:///home/marceleno/nti/jenkins_labs/lab3/Jenkinsfile))

```groovy
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
}
```
