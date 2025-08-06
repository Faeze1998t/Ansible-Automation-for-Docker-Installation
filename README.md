# Ansible-Automation-for-Docker-Installation
Hi everyone! I'm Faeze 👋
This repository provides a clean and secure way to Verify Ansible SSH connectivity to a remote Ubuntu server, Install Docker and Docker Compose Use Ansible Vault to keep credentials secure
🔐 Use Ansible Vault to keep credentials secure


## 🚀 Ansible Automation for Docker Installation

This repository helps you:

* Securely manage SSH access with Ansible Vault
* Test connection to remote Ubuntu servers
* Install Docker and Docker Compose using Ansible

---

## 📦 Prerequisites

* Ansible installed (you can use an Ansible container or virtual environment)
* A remote Ubuntu 24.04 server with a non-root sudo-enabled user
* SSH key-based authentication

---

## 🔐 Step 1: Add Encrypted Inventory File

Inside your Ansible environment, create or edit the `production.ini` inventory using Ansible Vault:

```bash
EDITOR=nano ansible-vault edit /ansible/inventory/production.ini
```

Add your host (example name, **replace with your actual values**):

```ini
[new_customer_servers]
<hostname> ansible_host=<IP_ADDRESS> ansible_user=<USERNAME> ansible_port=22
```

> ❗ **Do not commit plaintext IPs, usernames, or passwords.**

---

## ✅ Step 2: Create a Connection Test Playbook

Create a simple connection check playbook:

```bash
nano playbooks/check_connection.yml
```

Paste:

```yaml
---
- name: Check SSH connection to user server
  hosts: new_customer_servers
  gather_facts: false

  tasks:
    - name: Test connection using Ansible ping module
      ansible.builtin.ping:
```

---

## 🔧 Step 3: Configure Remote User for Ansible Access

On the **remote server**, allow the user to use `sudo` without a password:

```bash
sudo visudo
```

Add this line at the end:

```bash
<username> ALL=(ALL) NOPASSWD:ALL
```

Then add the user to the `sudo` group:

```bash
sudo usermod -aG sudo <username>
```

---

## 🔑 Step 4: Copy SSH Key to Remote Host

From your Ansible machine:

```bash
ssh-copy-id -i ~/.ssh/ansible_id_rsa.pub -p 22 <username>@<remote_ip>
```

---

## ▶️ Step 5: Run Connection Test

```bash
ansible-playbook -i /ansible/inventory/production.ini /ansible/playbooks/check_connection.yml --ask-vault-pass
```

Sample output:

```text
PLAY [Check SSH connection to user server] ******************************************************

TASK [Test connection using Ansible ping module] ************************************************
ok: [<hostname>]

PLAY RECAP **************************************************************************************
<hostname>     : ok=1  changed=0  unreachable=0  failed=0
```

 

## 🐳 Optional: Install Docker + Docker Compose

```
---
- name: Install Docker and Docker Compose on Ubuntu 24.04
  hosts: new_customer_servers
  become: true

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Install required packages for Docker
      apt:
        name:
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
          - software-properties-common
        state: present

    - name: Add Docker's official GPG key
      ansible.builtin.apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Set up the Docker repository
      ansible.builtin.apt_repository:
        repo: "deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_lsb.codename | lower }} stable"
        state: present
        filename: docker

    - name: Update apt cache after adding Docker repo
      apt:
        update_cache: yes

    - name: Install Docker Engine and Compose plugin
      apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-compose-plugin
        state: latest

    - name: Ensure Docker is started and enabled
      systemd:
        name: docker
        state: started
        enabled: true

    - name: Show Docker version
      command: docker --version
      register: docker_version
      changed_when: false

    - name: Show Docker Compose version
      command: docker compose version
      register: compose_version
      changed_when: false

    - name: Display Docker version
      debug:
        msg: "{{ docker_version.stdout }}"

    - name: Display Docker Compose version
      debug:
        msg: "{{ compose_version.stdout }}"

```

 

## 🛑 Security Notice

* Never publish real IPs, usernames, or passwords.
* Always use Ansible Vault for credentials.
* Use SSH key authentication, not plaintext passwords.

