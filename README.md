# CIFS-NetShare-with-Vault-Ansible
# Vault and Ansible on Rocky Linux

This guide provides step-by-step instructions to install and configure HashiCorp Vault and Ansible on Rocky Linux to securely manage secrets and automate tasks, including mounting a CIFS share.

## Prerequisites

- Rocky Linux server
- Root or sudo access

## Step 1: Install HashiCorp Vault

### 1.1 Add the HashiCorp repository

```sh
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
1.2 Install Vault

sudo yum install -y vault
1.3 Start Vault in development mode
Note: Development mode is not secure and should not be used in production.


vault server -dev -dev-listen-address="127.0.0.1:8200" &
export VAULT_ADDR='http://127.0.0.1:8200'
1.4 Login to Vault with the root token

vault login <Root Token>
Replace <Root Token> with the actual token provided when you started Vault in dev mode.

Step 2: Store CIFS credentials in Vault

vault kv put secret/cifs username=ogcinformatique password=ogcadm
Step 3: Install Ansible
3.1 Enable the EPEL repository

sudo yum install -y epel-release
3.2 Install Ansible

sudo yum install -y ansible
3.3 Verify the Ansible installation

ansible --version
Step 4: Configure Ansible to use Vault
4.1 Install the HashiCorp Vault Ansible collection

ansible-galaxy collection install community.hashi_vault
4.2 Create the Ansible configuration file
Create an ansible.cfg file in your project directory:


[defaults]
inventory = ./hosts
collections_paths = ~/.ansible/collections:/usr/share/ansible/collections
4.3 Create an inventory file
Create a hosts file in your project directory:


localhost ansible_connection=local
4.4 Create the Ansible playbook
Create a mount_cifs.yml file in your project directory:


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
      command: "mount -t cifs -o username={{ cifs_username }},password={{ cifs_password }},vers=2.0 //dc3-fic-wp1.domandpc.fr/Lecteurs/DIR/SSI/COMMUN/DEPOT/uploads /var/www/uploads"
4.5 Run the Ansible playbook

ansible-playbook -i hosts mount_cifs.yml

Conclusion
Following these steps, you will have installed and configured HashiCorp Vault and Ansible on Rocky Linux. You will also be able to securely manage CIFS credentials and automate the process of mounting a CIFS share using Ansible.

css
Copier le code

This `README.md` file provides a clear, step-by-step guide to setting up HashiCorp Vau
