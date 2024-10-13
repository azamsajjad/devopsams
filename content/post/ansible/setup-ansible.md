+++
title = 'Setup & Configure Ansible'
date = 2024-10-12T09:58:37Z
draft = false
description = "Sample article showcasing basic Markdown syntax and formatting for HTML elements."
featured = false
tags = [
    "ansible",
    "linux"
]
categories = ["ansible"]
thumbnail = "images/building.png"
+++
# Ansible Control Node Installation & Configuration

This guide walks through the steps for installing and configuring an Ansible control node, managing static and dynamic inventories, and defining groups of hosts.

## 1. Install Required Packages

When setting up an Ansible control node, ensure that the necessary packages are installed. Depending on your environment, follow the appropriate steps below.

### Installing Ansible from a Repository
```bash
# Search for ansible in the package manager
yum search ansible 

# List available repositories and find the Ansible repository
subscription-manager repos --list | grep ansible

# Enable the Ansible repository if needed (e.g., version 2.8)
subscription-manager repos --enable ansible-2.8-for-rhel-8-x86_64-rpms

# Install Ansible
yum install -y ansible
Installing Ansible from Source
```
```bash
# Clone the Ansible repository from GitHub (version 2.15 in this case)
git clone --single-branch --branch stable-2.15 https://github.com/ansible/ansible.git ansible

# Install dependencies
sudo yum install python3-pip
pip3 install --user -r ~/ansible/requirements.txt
```
2. Configuration & Static Inventory
Ansible Configuration File (ansible.cfg)
Create or modify the ansible.cfg file to manage Ansible's settings.

```bash
# Copy an example configuration file to /etc/ansible/
cp ~/git/ansible/examples/ansible.cfg /etc/ansible/ansible.cfg 
cp ~/git/ansible/examples/hosts /etc/ansible/hosts

# Or generate a new ansible.cfg file
ansible-config init --disable > ansible.cfg 
ansible-config list  # List available configuration options
Example ansible.cfg File
```ini
[defaults]
interpreter_python = auto
inventory = /home/rupert/ansible/inventory/inv.ini
roles_path = /etc/ansible/roles:/home/rupert/ansible/roles
Order of Preference for ansible.cfg:
```
* ANSIBLE_CONFIG environment variable
* ansible.cfg in the current working directory
* ansible.cfg in the user's home directory
* /etc/ansible/ansible.cfg
Creating a Static Host Inventory File
```bash
# Default inventory file is located at /etc/ansible/hosts
# Example of static inventory file content

[web]
192.168.100.101
192.168.100.102

[rhel_servers]
server1.example.com
server2.example.com

# Define hosts in ranges (IPs or hostnames)
192.168.100.[1:20]
server[1:9].example.com

# Define children groups
[test:children]
web
rhel_servers
```

Verify the Inventory
```bash
# List hosts from the web group
ansible web --list-hosts -i ~/ansible/inventory/inv.ini

# List all hosts
ansible all --list-hosts -i ~/ansible/inventory/inv.ini
```
3. Dynamic Inventory
Setting Up AWS Inventory
To manage dynamic AWS inventories, install required packages and set up Ansible for AWS EC2.

```bash
# Install necessary dependencies
yum install ansible-core python3-pip python3-devel
pip3 install --upgrade ansible
pip3 install boto3

# Install the AWS collection
ansible-galaxy collection install amazon.aws

# Clone ec2.py script and config file
git clone https://github.com/ansible/ansible-examples.git
```
Modify the ec2.ini file as needed (e.g., line 64 -> set elasticache to false)
Configure Dynamic Inventory in ansible.cfg
```ini
[defaults]
inventory = /home/ansible/aws/ec2.py
remote_user = rupert
ask_pass = false
private_key_file = /home/ansible/aws/ansible-key-pair.pem

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false
Example of Dynamic Inventory in AWS
```
```bash
# Test dynamic inventory
ansible all --list-hosts 

# Example output
52.207.214.10
44.208.34.168
```
### Managing Groups in Dynamic Inventory
The ec2.py script, along with its configuration file ec2.ini, is used to limit the scope of Ansible's reach within AWS. You can specify regions, instance tags, and roles that Ansible should find using AWS tags.

```bash
# List inventory in graph format
ansible-inventory --graph

# Grouping hosts using tags in playbook
- name: Example Playbook
  hosts: tag_group_web
  tasks:
    - name: Ping web servers
      ansible.builtin.ping:
Example Output from ansible-inventory --graph
```
```bash
@all:
  |--@ungrouped:
  |  |--localhost
  |--@vmservers:
  |  |--server1
  |  |--server2
  |--@rhel1:
  |  |--server1
  |--@rhel2:
  |  |--server2
Copy the SSH Key
```
```bash
# Copy ansible_key.pem to the AWS folder to access instances
cp ~/keys/ansible_key.pem /home/ansible/aws/
Additional Configuration in ansible.cfg
ini
[defaults]
inventory = /home/ansible/aws/ec2.py
remote_user = rupert
ask_pass = false
private_key_file = /home/ansible/aws/ansible-key-pair.pem

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false
Test with a Ping Command
```
```bash
ansible all -m ping
Managing Groups in Dynamic Inventory
You can manage groups and limit the scope of the dynamic inventory using the ec2.py script. Modify the ec2.ini file to specify the regions, instance tags, or roles.

```
```bash
# Example playbook with AWS EC2 tags
- name: Example Playbook with Groups
  hosts: tag_group_web
  tasks:
    - name: Ping web servers
      ansible.builtin.ping:

ansible-inventory --graph
```
This setup allows you to manage both static and dynamic inventories effectively on an Ansible control node.



* Check Ansible Configuration

```bash
ansible-config dump --only-changed  # Shows only changed configurations
```

* Testing Connectivity

```bash
ansible all -m ping  # Test connectivity with all hosts
```
* View Inventory Graph

```bash
ansible-inventory --graph  # View a visual representation of the inventory
```

By following these steps, you will have a fully functional Ansible control node with both static and dynamic inventories. You can now manage your infrastructure efficiently using Ansible's powerful automation capabilities.