# Server management

This playbook shows how to manage servers using Ansible.

## Prerequisites

First, we have to have python 3.8+ and ansible installed.

```bash
# Verify python version
python --version
# Output: Python 3.10.10

# Verify ansible
pip show ansible

# If not installed, install ansible
pip install ansible
```

## Setup hosts where the playbook will be executed

In order for the playbook to work, we have to ensure the following:

- The hosts must allow ssh connections from ansible controller
- A user with passwordless sudo (in order to execute ansible commands)
- The hosts must install python 3.8+ and ansible

If ssh connections are ensured and all setup are done, we can skip to next section. Otherwise, follow the following steps to setup the hosts.

**Note:** to do the following steps, you have to have root access

Fist step, create ansible user on the hosts.

```bash
# Fist, ssh to the host and run the following command
adduser ansible -uid 1100 # it is recommended to specify the uid
# After that, enter the password for the user and others information

# Verify if user created successfully
cat /etc/passwd | grep ansible
# Output: ansible:x:1100:1100:Ansible,,,:/home/ansible:/bin/bash

# Allow passwordless sudo, run the following command to edit /etc/sudoers file
sudo visudo

# Then, add the following line
ansible ALL=(ALL:ALL) NOPASSWD:ALL

# Then, login as the ansible user and verify if the user has passwordless sudo
sudo whoami
# Output: root
# Note that if setup is done correctly, you will not be prompted for password
```

Repeat the same steps for the other hosts.

Next, we need to setup ssh with ssh-key. First, on ansible controller host, if you not have ssh-key, generate it first:

```bash
ssh-keygen -t ed25519 -C "This key is use for ansible"
# You will be prompted for password for the key and where to store it, leave it as default


# Start SSH agent
eval "$(ssh-agent -s)"

# Add the created ssh key
ssh-add ~/.ssh/id_ed25519
# If this key has passsword, you will be prompted for password
```

Then, copy ssh key to the hosts:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519 ansible@<host_ip>
# For example: ssh-copy-id -i ~/.ssh/id_ed25519 ansible@192.168.20.156
# Enter the password for ansible user on the host

# Verify if ssh key copied successfully
ssh -i ~/.ssh/id_ed25519 ansible@<host_ip>
# You should be able to ssh without password
```

Then, verify python and ansible on the hosts:

```bash
# Verify python version
python --version
# Output: Python 3.10.10

# Verify ansible
pip show ansible

# If not installed, install ansible
pip install ansible
```

Final step is to add the hosts to the ansible inventory file.

```bash
# Open inventoty file and add the following line
[my_hosts]
host1 ansible_host=<host1_ip> ansible_user=ansible
host2 ansible_host=<host2_ip> ansible_user=ansible

# Example
[gpu_hosts]
gpu_156 ansible_host=192.168.20.156 ansible_user=ansible
gpu_170 ansible_host=192.168.20.170 ansible_user=ansible
```

Finally, test the playbook by running a script on the hosts:

```bash
ansible-playbook playbooks/run_script.yml
```

## Run the playbook

### Create user on multiple hosts

```bash
ansible-playbook playbooks/create_gpu_account.yml \
    -e hosts=<hosts_in_inventory> \
    -e username=<username> \
    -e uid=<uid> \
    -e user_state=present # or absent to delete user

# Example
ansible-playbook playbooks/create_gpu_account.yml \
    -e hosts=gpu_hosts \
    -e username=khiemle2409 \
    -e uid=1111 \
    -e user_state=present # or absent to delete user
```
