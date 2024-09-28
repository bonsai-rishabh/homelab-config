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
- [License](#license)
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

### 2. Set Up Ansible\n\nEnsure Ansible is installed on your control node:

```bash
sudo apt-get update\nsudo apt-get install -y ansible
```
Alternatively, for other Linux distributions or macOS, refer to the [Ansible installation guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).\n\n### 3. Configure Ansible Inventory and Variables\n\n- **Inventory File**: Edit `ansible/inventory/hosts` to define your homelab hosts.\n\n  ```ini\n  [zimaboard]\n  zimaboard1 ansible_host=192.168.1.10\n\n  [nfs_clients]\n  client1 ansible_host=192.168.1.11\n  client2 ansible_host=192.168.1.12\n\n  [all:vars]\n  ansible_user=your_user\n  ansible_ssh_private_key_file=~/.ssh/id_rsa\n  ```\n\n  Replace `your_user` with the username you use to SSH into your homelab servers, and ensure that the SSH key file path is correct.\n\n- **Group Variables**: Customize variables in `ansible/inventory/group_vars/`.\n\n  - **File**: `ansible/inventory/group_vars/all.yml`\n\n    ```yaml\n    ntp_servers:\n      - 0.pool.ntp.org\n      - 1.pool.ntp.org\n      - 2.pool.ntp.org\n\n    # Add other global variables here\n    ```\n\n  - **File**: `ansible/inventory/group_vars/zimaboard.yml`\n\n    ```yaml\n    network_interface: eth0\n    server_port: 8080\n    enable_feature_x: true\n    additional_settings: |\n      setting1 = value1\n      setting2 = value2\n\n    # Add Zimaboard-specific variables here\n    ```\n\n### 4. Prepare the PXE Boot Environment\n\n- **Configure TFTP Server**: Set up your TFTP server using the configurations in `configs/tftp/`.\n\n  - Copy `tftp.conf` to your TFTP server configuration directory (e.g., `/etc/`).\n\n- **Configure DHCP Server**: Use `configs/dhcp/dhcpd.conf` for DHCP settings to support PXE booting.\n\n  - Replace existing `dhcpd.conf` with the one provided or merge settings as necessary.\n\n- **Deploy PXE Files**: Use the `scripts/deploy_pxe.sh` script to copy PXE boot files to the TFTP server.\n\n  ```bash\n  ./scripts/deploy_pxe.sh\n  ```\n\n  - Ensure the script has executable permissions: `chmod +x scripts/deploy_pxe.sh`.\n\n- **Set Up HTTP Server**: Serve the Kickstart files and any repositories via HTTP.\n\n  - Place the Kickstart files in your web server's root directory or ensure the symlinks in `http/kickstarts/` point to the correct location.\n\n### 5. Configure GitHub Actions with Self-Hosted Runners\n\n- **Set Up Self-Hosted Runner**: Follow GitHub's [official guide](https://docs.github.com/en/actions/hosting-your-own-runners/adding-self-hosted-runners) to set up a self-hosted runner on your Ansible execution node.\n\n  - Ensure the runner has the necessary labels (e.g., `self-hosted`, `ansible`).\n\n- **Configure Workflows**: Check the workflows in `.github/workflows/` and adjust them if necessary.\n\n  - **CI Workflow**: `.github/workflows/ci.yml`\n  - **CD Workflow**: `.github/workflows/cd.yml`\n\n- **Set Secrets**: Add necessary secrets in your GitHub repository settings.\n\n  - Go to `Settings` > `Secrets and variables` > `Actions`.\n\n  - Add `ANSIBLE_VAULT_PASSWORD` and any other required secrets.\n\n### 6. Run Ansible Playbooks\n\n- **Test the Configuration**\n\n  Before deploying to all hosts, you can run a dry-run to test the playbooks:\n\n  ```bash\n  ansible-playbook ansible/playbooks/site.yml --check --diff\n  ```\n\n- **Deploy Configuration**\n\n  To deploy the configurations:\n\n  ```bash\n  ansible-playbook ansible/playbooks/site.yml\n  ```\n\n  - If you have encrypted variables with Ansible Vault:\n\n    ```bash\n    ansible-playbook ansible/playbooks/site.yml --ask-vault-pass\n    ```\n\n    Or use the vault password file:\n\n    ```bash\n    ansible-playbook ansible/playbooks/site.yml --vault-password-file ~/.ansible/vault_pass.txt\n    ```\n\n### 7. Automate with Scripts\n\n- **Manage Ansible Playbooks**: Use the `scripts/manage_ansible.sh` script to simplify running playbooks.\n\n  ```bash\n  ./scripts/manage_ansible.sh site.yml\n  ```\n\n  - Ensure the script is executable: `chmod +x scripts/manage_ansible.sh`.\n\n- **Deploy PXE Configurations**: Use the `scripts/deploy_pxe.sh` script to deploy PXE boot configurations.\n\n### 8. Monitor CI/CD Pipelines\n\n- **GitHub Actions**: Monitor the CI/CD workflows under the `Actions` tab in your GitHub repository.\n\n  - **CI Workflow**: Runs on pushes and pull requests to `main`.\n  - **CD Workflow**: Runs when code is merged into `main`.\n\n- **Workflow Logs**: Review logs for any errors or warnings.\n\n---\n\n## Configuration Management Strategy\n\n### Ansible Roles and Playbooks\n\n- **Roles**: Located in `ansible/roles/`, roles encapsulate tasks, variables, templates, and files.\n\n  - **Common Role**: General configurations applied to all hosts.\n\n  - **Zimaboard Role**: Specific configurations for Zimaboard servers.\n\n  - **NFS Client Role**: Configurations for NFS client setups.\n\n- **Playbooks**: Located in `ansible/playbooks/`, playbooks orchestrate roles across hosts.\n\n  - **site.yml**: Main playbook that includes all roles.\n\n  - **zimaboard.yml**: Playbook specific to Zimaboard servers.\n\n### Jinja2 Templates\n\n- **Templates**: Located in `templates/` directories within roles.\n\n  - Used to generate configuration files dynamically based on variables.\n\n  - Examples:\n\n    - `ntp.conf.j2`: Template for NTP configuration.\n\n    - `zimaboard_config.j2`: Template for Zimaboard-specific configurations.\n\n### Variables and Secrets Management\n\n- **Group Variables**: Located in `ansible/inventory/group_vars/`.\n\n  - Define variables for groups of hosts.\n\n- **Host Variables**: Can be added in `ansible/inventory/host_vars/`.\n\n  - Define variables for individual hosts.\n\n- **Ansible Vault**: Used to encrypt sensitive variables.\n\n  - **Encrypt a file**:\n\n    ```bash\n    ansible-vault encrypt ansible/inventory/group_vars/all/vault.yml\n    ```\n\n  - **Edit an encrypted file**:\n\n    ```bash\n    ansible-vault edit ansible/inventory/group_vars/all/vault.yml\n    ```\n\n  - **Decrypt a file**:\n\n    ```bash\n    ansible-vault decrypt ansible/inventory/group_vars/all/vault.yml\n    ```\n\n  - **Use vault password file**:\n\n    - Create a file with the vault password (e.g., `~/.ansible/vault_pass.txt`).\n\n    - Set appropriate permissions: `chmod 600 ~/.ansible/vault_pass.txt`.\n\n---\n\n## CI/CD Pipeline with GitHub Actions\n\n### Workflows\n\n- **CI Workflow (`ci.yml`)**: Performs linting, syntax checking, and testing.\n\n  - **Triggers**: On push or pull request to `main`.\n\n  - **Jobs**:\n\n    - **Checkout Code**: Uses `actions/checkout@v3`.\n\n    - **Set Up Python Environment**: Uses `actions/setup-python@v4`.\n\n    - **Install Dependencies**:\n\n      ```bash\n      python -m pip install --upgrade pip\n      pip install ansible ansible-lint yamllint\n      ansible-galaxy collection install community.general\n      ```\n\n    - **Run YAML Lint**:\n\n      ```bash\n      yamllint ansible/\n      ```\n\n    - **Run Ansible Lint**:\n\n      ```bash\n      ansible-lint ansible/\n      ```\n\n    - **Perform Syntax Check**:\n\n      ```bash\n      ansible-playbook ansible/playbooks/site.yml --syntax-check\n      ```\n\n    - **Test Playbooks (Dry Run)**:\n\n      ```bash\n      ansible-playbook ansible/playbooks/site.yml --check --diff --limit test_hosts\n      ```\n\n- **CD Workflow (`cd.yml`)**: Deploys configurations to the homelab.\n\n  - **Trigger**: On push to `main` after merging.\n\n  - **Jobs**:\n\n    - **Checkout Code**\n\n    - **Set Up Python Environment**\n\n    - **Install Dependencies**\n\n    - **Run Ansible Playbook**:\n\n      ```bash\n      ansible-playbook ansible/playbooks/site.yml --vault-password-file <(echo $ANSIBLE_VAULT_PASSWORD)\n      ```\n\n### Self-Hosted Runners\n\n- **Setup**: Follow GitHub's guide to add a self-hosted runner.\n\n- **Labels**: Assign labels like `self-hosted`, `ansible` to target the runner in workflows.\n\n- **Security**: Ensure the runner is secured and has limited access as necessary.\n\n### Secrets Management\n\n- **GitHub Secrets**: Store secrets in your repository settings.\n\n  - Example: `ANSIBLE_VAULT_PASSWORD`\n\n- **Access in Workflows**: Use the `secrets` context.\n\n  ```yaml\n  env:\n    ANSIBLE_VAULT_PASSWORD: ${{ secrets.ANSIBLE_VAULT_PASSWORD }}\n  ```\n\n### Triggering Workflows\n\n- **CI Workflow**: Runs automatically on pushes or pull requests.\n\n- **CD Workflow**: Runs when code is merged into `main`.\n\n- **Manual Triggers**: Workflows can also be triggered manually from the GitHub Actions tab.\n\n### Monitoring Workflows\n\n- **Actions Tab**: View the status of workflows.\n\n- **Logs**: Examine logs for detailed information on workflow runs.\n\n- **Notifications**: Configure notifications for workflow events.\n\n---\n\n## Additional Resources\n\n- **Host Setup Guides**: Detailed setup instructions in `docs/host_setup_guides/`.\n\n  - Example: `zimaboard.md` for Zimaboard setup.\n\n- **Ansible Documentation**: [Ansible Official Documentation](https://docs.ansible.com/)\n\n- **GitHub Actions Documentation**: [GitHub Actions Documentation](https://docs.github.com/en/actions)\n\n- **Jinja2 Template Documentation**: [Jinja2 Templates](https://jinja.palletsprojects.com/)\n\n---\n\n## Contributing\n\nContributions are welcome! Please follow these guidelines:\n\n- **Fork the Repository**: Create a personal fork of the repository.\n\n- **Create a Feature Branch**: For your changes.\n\n  ```bash\n  git checkout -b feature/your-feature-name\n  ```\n\n- **Make Changes**: Commit your changes with clear messages.\n\n- **Push to Your Fork**:\n\n  ```bash\n  git push origin feature/your-feature-name\n  ```\n\n- **Open a Pull Request**: Describe your changes and submit the PR.\n\n- **Ensure CI Passes**: All workflows must pass before merging.\n\n---\n\n## License\n\n[Your chosen license here]\n\n---\n\n## Contact\n\nFor any questions or support, please contact [Your Name](mailto:your.email@example.com).\n\n---\n"


