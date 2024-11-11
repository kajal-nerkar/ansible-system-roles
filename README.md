# Ansible System Automation

An Ansible project designed for automating system configurations on remote servers, focusing on:

- **Local User Account Management**: Creating and managing local user accounts with customizable options.
- **SSH Daemon Configuration**: Ensuring secure SSH settings for user authentication.

## Project Objectives

The main objective of this project is to provide a reusable, modular automation solution for system administrators. It enables them to:

1. **Manage Local User Accounts**: Automatically create, modify, or delete local users with specified configurations such as shell, user ID, home directory, and expiry date.
2. **Configure SSH for Security**: Set up SSH daemon options to enforce secure settings, including password authentication and root login policies.

This project is implemented using Ansible, which ensures that the configurations are **idempotent**—meaning they will be applied only if necessary, without redundant changes.

## Project Structure:
```
ansible-system-automation/
├── README.md                       # Project documentation with objectives, setup guide, and usage
├── inventory.yml                   # Inventory file specifying target hosts for the playbooks
├── playbook.yml                    # Main Ansible playbook that calls the roles
├── galaxy.yml                      # Metadata for the project, used if publishing to Ansible Galaxy
├── roles/
│   ├── local_accounts/
│   │   ├── defaults/
│   │   │   └── main.yml            # Default variables for the local_accounts role
│   │   ├── files/                  # Files to copy to remote hosts
│   │   ├── handlers/
│   │   │   └── main.yml            # Handlers for local_accounts role (e.g., service restart)
│   │   ├── meta/
│   │   │   └── main.yml            # Metadata and dependencies for local_accounts role
│   │   ├── tasks/
│   │   │   └── main.yml            # Main tasks for the local_accounts role
│   │   ├── templates/              # Jinja2 templates for dynamic configuration
│   │   └── vars/
│   │       └── main.yml            # Role-specific variables
│   └── ssh_config/
│       ├── defaults/
│       │   └── main.yml            # Default variables for ssh_config role
│       ├── handlers/
│       │   └── main.yml            # Handlers for ssh_config role
│       ├── meta/
│       │   └── main.yml            # Metadata and dependencies for ssh_config role
│       ├── tasks/
│       │   └── main.yml            # Main tasks for the ssh_config role
│       └── templates/              # Optional templates for ssh_config role
└── tests/
    ├── inventory                   # Test inventory file
    └── test_playbook.yml           # Playbook to test the roles in the project

```

## Prerequisites:
1. WSL
2. Ansible
3. VM on AWS

## Deployment Steps:
### 1. Clone Repository:
```
git clone https://github.com/kajal-nerkar/ansible-system-roles.git
cd ansible-system-roles
```
### 2. Configure the Inventory File
```
Example inventory.yml:
all:
  hosts:
    target_server:
      ansible_host: <your-remote-server-ip>
      ansible_user: <your-remote-user>
      ansible_ssh_private_key_file: ~/.ssh/your-key.pem

Replace:
    <your-remote-server-ip> with the IP address of your target server.
    <your-remote-user> with the SSH user on the target server.
    ~/.ssh/your-key.pem with the path to your private SSH key which was downloaded by aws ec2

```
### 3. Run the Playbook
```
ansible-playbook -i inventory.yml playbook.yml --ask-become-pass
```

## Verify the Deployment:

### 1. Connect to the Target Server:
```
ssh -i ~/.ssh/your-key.pem <your-remote-user>@<your-remote-server-ip>

Replace <your-remote-user> and <your-remote-server-ip> with the actual SSH username and IP address of your target server.
```

### 2. Verify Local User Accounts 
```
cat /etc/passwd | grep <username>

id local_adm
id local_log

Expected Output:
uid=38000087(local_adm) gid=1001(local_adm) groups=1001(local_adm)
uid=38000088(local_log) gid=1002(local_log) groups=1002(local_log)

Check Home Directory and Permissions:
ls -ld /home/local_adm
ls -ld /home/local_log

Expected Output:
drwxr-x--- 2 local_adm local_adm 4096 Nov 11 11:58 /home/local_adm
drwxr-x--- 2 local_log local_log 4096 Nov 11 12:13 /home/local_log

```
### 3. Verify Account Expirey Date:
```
sudo chage -l local_adm

Expected Output:
Last password change                                    : Nov 11, 2024
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7


sudo chage -l local_log

Expected Output:
Last password change                                    : Nov 11, 2024
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Dec 30, 2024
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99
```
### 4. Verify SSH Configuration 
```
Check SSH Daemon Configuration:
sudo grep -E '^PasswordAuthentication|^PermitEmptyPasswords|^PermitRootLogin' /etc/ssh/sshd_config

Expected Output:
PasswordAuthentication yes
PermitEmptyPasswords no
PermitRootLogin no

Verify SSH Service Status:
sudo systemctl status ssh

Expected Output:
The SSH service should be active (running).
```

### 5. Verify Passwordless SSH Access
```
ssh -i ~/.ssh/local_adm_key local_adm@<your-remote-server-ip>

Expected Outcome:
You should be able to log in without being prompted for a password.
```

### 6. Test Playbook Idempotency
```
ansible-playbook -i inventory.yml playbook.yml --ask-become-pass

Expected Output:

All tasks should show as ok rather than changed, indicating no further modifications were made.
```
