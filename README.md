# Ansible-Automation-for-Docker-Installation

Here’s a clean and professional `README.md` for your GitHub repository that documents how to check Ansible SSH connectivity and install Docker + Docker Compose on a remote Ubuntu server, **without exposing any IPs or passwords**.

---

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

To install Docker and Compose using a dedicated playbook, see [`docker-installation.yml`](./playbooks/docker-installation.yml) (create this file based on the main playbook logic).

 

## 🛑 Security Notice

* Never publish real IPs, usernames, or passwords.
* Always use Ansible Vault for credentials.
* Use SSH key authentication, not plaintext passwords.

