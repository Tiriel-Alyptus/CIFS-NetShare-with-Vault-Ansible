# Vault and Ansible on Ubuntu

This guide provides step-by-step instructions to install and configure HashiCorp Vault and Ansible on Ubuntu to securely manage secrets and automate tasks, including mounting a CIFS share.

## Prerequisites

- Ubuntu server
- Root or sudo access

## Step 1: Install HashiCorp Vault

### 1.1 Add the HashiCorp repository

```sh
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
```

```sh
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
```

### 1.2 Install Vault

```sh
sudo apt-get update && sudo apt-get install -y vault
```

### 1.3 Start Vault in development mode
Note: Development mode is not secure and should not be used in production.

```sh
vault server -dev -dev-listen-address="127.0.0.1:8200" &
export VAULT_ADDR='http://127.0.0.1:8200'
```

### 1.4 Login to Vault with the root token

```sh
vault login <Root Token>
```

Replace <Root Token> with the actual token provided when you started Vault in dev mode.

## Step 2: Store CIFS credentials in Vault

```sh
vault kv put secret/cifs username=yourusername password=yourpassword
```

## Step 3: Install Ansible
### 3.1 Install Ansible

```sh
sudo apt-get install -y ansible
```

### 3.2 Verify the Ansible installation

```sh
ansible --version
```

## Step 4: Configure Ansible to use Vault
### 4.1 Install the HashiCorp Vault Ansible collection

```yml
ansible-galaxy collection install community.hashi_vault
```

### 4.2 Create the Ansible configuration file
Create an ansible.cfg file in your project directory:

```yml
[defaults]
inventory = ./hosts
collections_paths = ~/.ansible/collections:/usr/share/ansible/collections
```

### 4.3 Create an inventory file
Create a hosts file in your project directory:

```yml
localhost ansible_connection=local
```

### 4.4 Create the Ansible playbook
Create a **mount_cifs.yml** file in your project directory:

```yml
---
- name: Mount CIFS Share
  hosts: localhost
  tasks:
    - name: Get CIFS credentials from Vault
      community.hashi_vault.vault_read:
        path: "secret/data/cifs"
        url: "http://127.0.0.1:8200"
        token: "<Root Token>"  # Replace with your actual Vault token
      register: cifs_secrets

    - name: Debug CIFS credentials
      debug:
        var: cifs_secrets

    - name: Set CIFS credentials
      set_fact:
        cifs_username: "{{ cifs_secrets.data.data.username }}"
        cifs_password: "{{ cifs_secrets.data.data.password }}"

    - name: Mount CIFS share
      become: yes
      command: "mount -t cifs -o username={{ cifs_username }},password={{ cifs_password }},vers=2.0 //windows.server.local/folder/to/share /var/example"
```

### 4.5 Run the Ansible playbook

```yml
ansible-playbook -i hosts mount_cifs.yml
```

## Conclusion
Following these steps, you will have installed and configured HashiCorp Vault and Ansible on Ubuntu. You will also be able to securely manage CIFS credentials and automate the process of mounting a CIFS share using Ansible.
