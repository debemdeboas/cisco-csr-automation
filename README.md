# Cisco section

This folder contains everything necessary to (barely) configure a Cisco CSR 1000V device.

## Methodology

### Provisioning the devices

The Cisco CSR device should be provisioned on the Cisco DevNet Sandbox for testing.
Connections to the sandbox are made using a VPN as required by Cisco themselves.

### Ansible

Following the Ansible guidelines, the tasks are split in two: a playbook on the root directory calling a role with a `main.yml` file.
The `cisco.ios` collection is used to interface with the device.

The `cisco_csr` role contains a `tasks/main.yml` file, which could be considered the playbook itself.
This file defines the actual tasks that are to be executed on the remote device.

First, the login banner is changed (defined in `group_vars/cisco.yml`).
Secondly, the users (defined in `group_vars/cisco_csr.yml`) are created with no password for demonstration purposes only.
Thirdly, the user passwords are set (also defined in the `cisco_csr` group variables file).
Lastly, a sample configuration template is applied.

#### The vault

Some secret variables are encrypted by `ansible-vault` and need to be decrypted during playbook execution.
To do so, edit the `.vault_pass` file with the vault password.

#### Inventory hierarchy

A role-based inventory hierarchy is used.
For example, a Cisco CSR device is in the group `cisco_csr`, which is a child of the `cisco` group.
By doing so, the device inherits all the `cisco` and `cisco_csr` group variables, facilitating further expansion into other `cisco` devices.

## Setup and run

Before running, the environment must be created:

```bash
# cd into this directory (if not already here)
cd cisco
# Create the virtual environment
python -m venv venv
# Activate the virtual environment
source venv/bin/activate
# Install the required Python packages 
pip install -r requirements.txt
# Install the required Ansible Galaxy collections
ansible-galaxy collection install -r requirements.yml
```

Running this playbook is fairly simple:

```bash
ansible-playbook -i inventory.ini configure_cisco_csr.yml --vault-pass-file .vault_pass
```

Command breakdown:
- `-i inventory.ini`: inventory file
- `configure_cisco_csr.yml`: the Cisco CSR configuration playbook file
- `--vault-pass-file .vault_pass`: the vault password file, used for decrypting encrypted variables.

The playbook also supports the following tags:

- `query_version`: query the CSR version, will never execute unless explicitly called
- `change_login_banner`: change the login banner
- `create_users`: create the users and change their passwords
- `configure_from_template`: push a Jinja2 template to the remote CSR device


