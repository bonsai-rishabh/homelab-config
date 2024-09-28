# Homelab Configuration Documentation

This document provides a comprehensive guide to the **Homelab Configuration Repository**, including the directory structure, setup instructions, configuration management strategy, CI/CD pipeline, and additional resources.

---

## Table of Contents

- [Introduction](#introduction)
- [Directory Structure](#directory-structure)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
  - [1. Clone the Repository](#1-clone-the-repository)
  - [2. Set Up Ansible](#2-set-up-ansible)
  - [3. Configure Ansible Inventory and Variables](#3-configure-ansible-inventory-and-variables)
  - [4. Prepare the PXE Boot Environment](#4-prepare-the-pxe-boot-environment)
  - [5. Configure GitHub Actions with Self-Hosted Runners](#5-configure-github-actions-with-self-hosted-runners)
- [Configuration Management Strategy](#configuration-management-strategy)
  - [Ansible Roles and Playbooks](#ansible-roles-and-playbooks)
  - [Jinja2 Templates](#jinja2-templates)
  - [Variables and Secrets Management](#variables-and-secrets-management)
- [CI/CD Pipeline with GitHub Actions](#cicd-pipeline-with-github-actions)
  - [Workflows](#workflows)
  - [Self-Hosted Runners](#self-hosted-runners)
  - [Secrets Management](#secrets-management)
  - [Triggering Workflows](#triggering-workflows)
  - [Monitoring Workflows](#monitoring-workflows)
- [Additional Resources](#additional-resources)
- [Contributing](#contributing)
- [Contact](#contact)

---

## Introduction

This repository contains all the necessary configuration files, Ansible playbooks, templates, and scripts to set up and manage a homelab environment. It leverages Ansible for configuration management, Jinja2 templates for dynamic configuration files, and GitHub Actions for CI/CD automation using self-hosted runners.

---

## Directory Structure

The repository is organized as follows:

<pre>
homelab-config/
├── ansible/
│   ├── inventory/
│   │   ├── group_vars/
│   │   │   ├── all.yml
│   │   │   ├── zimaboard.yml
│   │   │   └── other_hosts.yml
│   │   └── hosts
│   ├── roles/
│   │   ├── common/
│   │   │   ├── tasks/
│   │   │   ├── templates/
│   │   │   └── files/
│   │   ├── zimaboard/
│   │   │   ├── tasks/
│   │   │   ├── templates/
│   │   │   └── files/
│   │   ├── nfs_client/
│   │   │   ├── tasks/
│   │   │   ├── templates/
│   │   │   └── files/
│   │   └── other_roles/
│   └── playbooks/
│       ├── site.yml
│       ├── zimaboard.yml
│       └── other_playbooks.yml
├── pxe_boot/
│   ├── tftpboot/
│   │   ├── pxelinux.cfg/
│   │   ├── images/
│   │   └── menu.c32
│   └── kickstart/
│       ├── ks_zimaboard.cfg
│       └── ks_other_hosts.cfg
├── http/
│   ├── kickstarts/
│   └── repo/
├── scripts/
│   ├── deploy_pxe.sh
│   ├── manage_ansible.sh
│   └── utilities/
├── docs/
│   ├── README.md
│   └── host_setup_guides/
├── configs/
│   ├── nfs/
│   ├── dhcp/
│   ├── tftp/
│   └── other_services/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
└── LICENSE
</pre>

---

## Prerequisites

- **Operating System**: Linux-based systems for the homelab servers and Ansible execution node.
- **Ansible**: Installed on the control node.
- **Git**: Installed for version control.
- **Python**: Version 3.x for running Ansible and scripts.
- **GitHub Account**: For hosting the repository and using GitHub Actions.
- **Self-Hosted Runner**: Configured on your Ansible execution node.

---

## Setup Instructions

### 1. Clone the Repository

```bash 
git clone https://github.com/yourusername/homelab-config.git\ncd homelab-config/
```

### 2. Set Up Ansible

Ensure Ansible is installed on your control node:

```bash
sudo apt-get update\nsudo apt-get install -y ansible
```
Alternatively, for other Linux distributions or macOS, refer to the [Ansible installation guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

### 3. Configure Ansible Inventory and Variables

- **Inventory File**: Edit `ansible/inventory/hosts` to define your homelab hosts.

  
```ini
[zimaboard]
  zimaboard1 ansible_host=192.168.1.10

  [nfs_clients]
  client1 ansible_host=192.168.1.11
  client2 ansible_host=192.168.1.12

  [all:vars]
  ansible_user=your_user
  ansible_ssh_private_key_file=~/.ssh/id_rsa
```

  Replace `your_user` with the username you use to SSH into your homelab servers, and ensure that the SSH key file path is correct.
  
- **Group Variables**: Customize variables in `ansible/inventory/group_vars/`.
- **File**: `ansible/inventory/group_vars/all.yml`
  ```yaml
      ntp_servers:
        - 0.pool.ntp.org
        - 1.pool.ntp.org
        - 2.pool.ntp.org

      # Add other global variables here
  ```

- **File**: `ansible/inventory/group_vars/zimaboard.yml`

    ```yaml
      network_interface: eth0
        server_port: 8080
        enable_feature_x: true
        additional_settings: |
          setting1 = value1
          setting2 = value2

        # Add Zimaboard-specific variables here
    ```
### 4. Prepare the PXE Boot Environment

- **Configure TFTP Server**: Set up your TFTP server using the configurations in `configs/tftp/`.
  
  - Copy `tftp.conf` to your TFTP server configuration directory (e.g., `/etc/`).
    
- **Configure DHCP Server**: Use `configs/dhcp/dhcpd.conf` for DHCP settings to support PXE booting.
  
    - Replace existing `dhcpd.conf` with the one provided or merge settings as necessary.
    - **Deploy PXE Files**: Use the `scripts/deploy_pxe.sh` script to copy PXE boot files to the TFTP server.
    
      ```bash
        ./scripts/deploy_pxe.sh
        ```
        - Ensure the script has executable permissions: `chmod +x scripts/deploy_pxe.sh`.
- **Set Up HTTP Server**: Serve the Kickstart files and any repositories via HTTP.

  - Place the Kickstart files in your web server's root directory or ensure the symlinks in `http/kickstarts/` point to the correct location.
### 5. Configure GitHub Actions with Self-Hosted Runners
- **Set Up Self-Hosted Runner**: Follow GitHub's [official guide](https://docs.github.com/en/actions/hosting-your-own-runners/adding-self-hosted-runners) to set up a self-hosted runner on your Ansible execution node.

  - Ensure the runner has the necessary labels (e.g., `self-hosted`, `ansible`).
  - **Configure Workflows**: Check the workflows in `.github/workflows/` and adjust them if necessary.
  
    - **CI Workflow**: `.github/workflows/ci.yml`
  
    - **CD Workflow**: `.github/workflows/cd.yml`

    - **Set Secrets**: Add necessary secrets in your GitHub repository settings.
    
      - Go to `Settings` > `Secrets and variables` > `Actions`.
        - Add `ANSIBLE_VAULT_PASSWORD` and any other required secrets.
### 6. Run Ansible Playbooks
- **Test the Configuration**
  Before deploying to all hosts, you can run a dry-run to test the playbooks:
    ```bash
      ansible-playbook ansible/playbooks/site.yml --check --diff
    ```
    
- **Deploy Configuration**
    To deploy the configurations:
    ```bash
    ansible-playbook ansible/playbooks/site.yml
    ```
    - If you have encrypted variables with Ansible Vault:
      ```bash
          ansible-playbook ansible/playbooks/site.yml --ask-vault-pass
      ```
      
      Or use the vault password file:

      ```bash
          ansible-playbook ansible/playbooks/site.yml --vault-password-file ~/.ansible/vault_pass.txt
      ```
### 7. Automate with Scripts

- **Manage Ansible Playbooks**: Use the `scripts/manage_ansible.sh` script to simplify running playbooks.
  ```bash
    ./scripts/manage_ansible.sh site.yml
  ```
  - Ensure the script is executable: `chmod +x scripts/manage_ansible.sh`.
- **Deploy PXE Configurations**: Use the `scripts/deploy_pxe.sh` script to deploy PXE boot configurations.

### 8. Monitor CI/CD Pipelines

- **GitHub Actions**: Monitor the CI/CD workflows under the `Actions` tab in your GitHub repository.

  - **CI Workflow**: Runs on pushes and pull requests to `main`.
  - **CD Workflow**: Runs when code is merged into `main`
  - **Workflow Logs**: Review logs for any errors or warnings.
---

## Configuration Management Strategy

### Ansible Roles and Playbooks

- **Roles**: Located in `ansible/roles/`, roles encapsulate tasks, variables, templates, and files.
  - **Common Role**: General configurations applied to all hosts.
  - **Zimaboard Role**: Specific configurations for Zimaboard servers.
  - **NFS Client Role**: Configurations for NFS client setups.
- **Playbooks**: Located in `ansible/playbooks/`, playbooks orchestrate roles across hosts.
  - **site.yml**: Main playbook that includes all roles.
  - **zimaboard.yml**: Playbook specific to Zimaboard servers.
### Jinja2 Templates

- **Templates**: Located in `templates/` directories within roles.

  - Used to generate configuration files dynamically based on variables.
    - Examples:
        - `ntp.conf.j2`: Template for NTP configuration.
        - `zimaboard_config.j2`: Template for Zimaboard-specific configurations.
### Variables and Secrets Management

- **Group Variables**: Located in `ansible/inventory/group_vars/`.
  - Define variables for groups of hosts.
- **Host Variables**: Can be added in `ansible/inventory/host_vars/`.
  - Define variables for individual hosts.
- **Ansible Vault**: Used to encrypt sensitive variables.
  - **Encrypt a file**:
      ```bash
        ansible-vault encrypt ansible/inventory/group_vars/all/vault.yml
      ```
  - **Edit an encrypted file**:
      ```bash
        ansible-vault edit ansible/inventory/group_vars/all/vault.yml
      ```

  - **Decrypt a file**:
      ```bash
        ansible-vault decrypt ansible/inventory/group_vars/all/vault.yml
      ```

  - **Use vault password file**:
      - Create a file with the vault password (e.g., `~/.ansible/vault_pass.txt`).
      - Set appropriate permissions: `chmod 600 ~/.ansible/vault_pass.txt`.
---
## CI/CD Pipeline with GitHub Actions
### Workflows
- **CI Workflow (`ci.yml`)**: Performs linting, syntax checking, and testing.
  - **Triggers**: On push or pull request to `main`.
  - **Jobs**:
     - **Checkout Code**: Uses `actions/checkout@v3`.
     - **Set Up Python Environment**: Uses `actions/setup-python@v4`.
     - **Install Dependencies**:
          ```bash
             python -m pip install --upgrade pip
             pip install ansible ansible-lint yamllint
             ansible-galaxy collection install community.general
          ```
     - **Run YAML Lint**:
          ```bash
            yamllint ansible
          ```
     - **Run Ansible Lint**:
          ```bash
             ansible-lint ansible/
          ```
     - **Perform Syntax Check**:
          ```bash
                ansible-playbook ansible/playbooks/site.yml --syntax-check
          ```
     - **Test Playbooks (Dry Run)**:
          ```bash
                ansible-playbook ansible/playbooks/site.yml --check --diff --limit test_hosts
          ```
- **CD Workflow (`cd.yml`)**: Deploys configurations to the homelab.
  - **Trigger**: On push to `main` after merging.
  - **Jobs**:
      - **Checkout Code**
      - **Set Up Python Environment**
      - **Install Dependencies**
      - **Run Ansible Playbook**:
          ```bash
              ansible-playbook ansible/playbooks/site.yml --vault-password-file <(echo $ANSIBLE_VAULT_PASSWORD)
          ```
### Self-Hosted Runners
- **Setup**: Follow GitHub's guide to add a self-hosted runner.
- **Labels**: Assign labels like `self-hosted`, `ansible` to target the runner in workflows.
- **Security**: Ensure the runner is secured and has limited access as necessary.

### Secrets Management
- **GitHub Secrets**: Store secrets in your repository settings.
  - Example: `ANSIBLE_VAULT_PASSWORD`
- **Access in Workflows**: Use the `secrets` context.
  ```yaml
    env:
      ANSIBLE_VAULT_PASSWORD: ${{ secrets.ANSIBLE_VAULT_PASSWORD }}
  ```
### Triggering Workflows
- **CI Workflow**: Runs automatically on pushes or pull requests.
- **CD Workflow**: Runs when code is merged into `main`.
- **Manual Triggers**: Workflows can also be triggered manually from the GitHub Actions tab.

### Monitoring Workflows
- **Actions Tab**: View the status of workflows.

- **Logs**: Examine logs for detailed information on workflow runs.

- **Notifications**: Configure notifications for workflow events.

---
## Additional Resources

- **Host Setup Guides**: Detailed setup instructions in `docs/host_setup_guides/`.

  - Example: `zimaboard.md` for Zimaboard setup.

- **Ansible Documentation**: [Ansible Official Documentation](https://docs.ansible.com/)

- **GitHub Actions Documentation**: [GitHub Actions Documentation](https://docs.github.com/en/actions)

- **Jinja2 Template Documentation**: [Jinja2 Templates](https://jinja.palletsprojects.com/)

---

## Contributing
Contributions are welcome! Please follow these guidelines:\

- **Fork the Repository**: Create a personal fork of the repository.

- **Create a Feature Branch**: For your changes.

```bash
git checkout -b feature/your-feature-name
```

- **Make Changes**: Commit your changes with clear messages.

- **Push to Your Fork**:
```bash
git push origin feature/your-feature-name
```

- **Open a Pull Request**: Describe your changes and submit the PR.
- **Ensure CI Passes**: All workflows must pass before merging.

---

## Contact

For any questions or support, please contact [Rishabh Bajpai](mailto:your.email@example.com).

---


