# 📘  Ansible EC2 Automation & VM Metrics**

## 📌 Overview

This project uses **Ansible** to automate package installation eg:(Docker, Maven), create directories, and collect **system metrics** (CPU, Memory, Disk) from multiple EC2 instances.  
It also uses the **AWS EC2 Dynamic Inventory Plugin** to automatically fetch host IP addresses based on EC2 tags.

---

## 📁 Architecture Overview

```
inventory/
├── aws_ec2.yaml    # Dynamic inventory configuration
├── playbook.yaml   # Installs packages (Docker, Maven)

group_vars/
├── os_ubuntu.yaml  # Ubuntu-specific tags
├── os_amazon.yaml  # Amazon Linux-specific tags
├── all.yaml        # smtp & email credentials

templates/
├── report_email_animated.html.j2 # HTML format for the hosts CPU and usage metrics

├── ansible.cfg      # separate environment to run ansible configuration
├── tag.sh           # tagging the running hosts with name web01 & 02 
├── copy_pub.sh      # copying the pub key from the master node to the host machines 
├── collect_metrics.yaml # collecting metrics like CPU usage, memory, and disk
├── playbook.yaml    # running collect_metrics.yaml & send_report.yaml
├── send_report.yaml # sending consolidated  report with the timestamp and email credentials

```

---

## 🌐 Executions
- Install Ansible on the Ansible master with AWS CLI and create the environment with ansible.cfg
- Tagging the hosts machines #tag.sh
Name=Environment, Values=dev
Name=os,Values=ubuntu #if the host is ubuntu
Name=os,Values=amazon #if the host is redhat distro
  
<img width="621" height="77" alt="image" src="https://github.com/user-attachments/assets/b56cefdd-c4f0-41c4-8bac-9e5649e30787" />

- Generate ssh-keygen for the master node and copy the master.pem file and injecting ssh public key into hosts #copy_pub.sh
- Run the ansible-inventory -i inventory/aws_ec2.yaml --graph to show discovered IP addresses
<img width="704" height="418" alt="image" src="https://github.com/user-attachments/assets/9234644a-e600-4ba4-9220-98c3da251e24" />

- pinging the ping pong output to the remote hosts, categorizing Ubuntu and Amazon hosts
- @os_ubuntu
  └── ec2-13-203-160-135.ap-south-1.compute.amazonaws.com
- @os_amazon
  └── ec2-13-127-79-63.ap-south-1.compute.amazonaws.com
<img width="1587" height="525" alt="image" src="https://github.com/user-attachments/assets/b9624be4-947d-49f8-96ed-513870590fcf" />


## 🚀 Running the Playbooks

### 1️⃣ Run setup (install Docker, Maven)

```
ansible-playbook -i inventory/aws_ec2.yaml playbook-setup.yaml
```

### 2️⃣ Run metrics collection

```
ansible-playbook -i inventory/aws_ec2.yaml playbook-metrics.yaml
```

### 3️⃣ Show discovered IP addresses

```
ansible-inventory -i inventory/aws_ec2.yaml --graph
```

Example:

```
@os_ubuntu
  └── ec2-13-203-160-135.ap-south-1.compute.amazonaws.com
@os_amazon
  └── ec2-15-207-222-108.ap-south-1.compute.amazonaws.com
```

---



- Dynamic inventory automatically updates when EC2 instances change  
- OS tagging (`os=ubuntu`, `os=amazon`) ensures correct package manager  
- Metrics playbook gives a clean consolidated VM health report  
- Setup playbook installs required tools across mixed OS environments  

Your `aws_ec2.yaml` automatically discovers EC2 instances using tags:

```yaml
plugin: amazon.aws.aws_ec2
regions:
  - ap-south-1
filters:
  tag:Environment: dev
  instance-state-name: running

compose:
  ansible_host: public_ip_address

keyed_groups:
  - key: tags.os
    prefix: os

vars:
  ansible_ssh_private_key_file: /home/ubuntu/masterkey.pem
```

### ✔ How OS grouping works

- EC2 tag: `os=ubuntu` → group: `os_ubuntu`
- EC2 tag: `os=amazon` → group: `os_amazon`

### ✔ Group variables

`group_vars/os_ubuntu.yaml`:

```yaml
ansible_user: ubuntu
```

`group_vars/os_amazon.yaml`:

```yaml
ansible_user: ec2-user
```

This ensures Ansible uses the correct SSH user per OS.

---

## 🐳 **Setup Playbook — Install Docker, Maven, Net-tools**

`playbook-setup.yaml`:

```yaml
---
- hosts: all
  become: yes

  tasks:

    - name: print message
      debug:
        msg: "Hello-Ansible Worlds"

    - name: create directory
      file:
        path: /home/ubuntu/test1
        state: directory

    # Ubuntu
    - name: install maven on Ubuntu
      apt:
        name: maven
        state: present
        update_cache: yes
      when: ansible_os_family == "Debian"

    - name: install docker on Ubuntu
      apt:
        name: docker.io
        state: present
        update_cache: yes
      when: ansible_os_family == "Debian"

    - name: install net-tools on Ubuntu
      apt:
        name: net-tools
        state: present
        update_cache: yes
      when: ansible_os_family == "Debian"

    # Amazon Linux
    - name: install maven on Amazon Linux
      yum:
        name: maven
        state: present
      when: ansible_os_family == "RedHat"

    - name: install docker on Amazon Linux
      yum:
        name: docker
        state: present
      when: ansible_os_family == "RedHat"

    - name: install net-tools on Amazon Linux
      yum:
        name: net-tools
        state: present
      when: ansible_os_family == "RedHat"
```

---

## 📊 **Metrics Playbook — CPU, Memory, Disk Usage**

Your metrics playbook:

- Installs `sysstat`
- Collects CPU via `mpstat`
- Collects memory usage
- Collects disk usage
- Builds a consolidated report
- Emails the report

Example output:

```
3 VMs | Avg CPU: 0.17% | Avg Mem: 35.68% | Avg Disk: 21.67%
```

---

## 🧪 Testing Connectivity

Run:

```
ansible all -i inventory/aws_ec2.yaml -m ping
```

Expected output:

```
ec2-xx-xx-xx-xx.ap-south-1.compute.amazonaws.com | SUCCESS => pong
```

---

## 🚀 Running the Playbooks

### 1️⃣ Run setup (install Docker, Maven)

```
ansible-playbook -i inventory/aws_ec2.yaml playbook-setup.yaml
```

### 2️⃣ Run metrics collection

```
ansible-playbook -i inventory/aws_ec2.yaml playbook-metrics.yaml
```

### 3️⃣ Show discovered IP addresses

```
ansible-inventory -i inventory/aws_ec2.yaml --graph
```

Example:

```
@os_ubuntu
  └── ec2-13-203-160-135.ap-south-1.compute.amazonaws.com
@os_amazon
  └── ec2-15-207-222-108.ap-south-1.compute.amazonaws.com
```

---

## 🏁 Final Notes

- Dynamic inventory automatically updates when EC2 instances change  
- OS tagging (`os=ubuntu`, `os=amazon`) ensures correct package manager  
- Metrics playbook gives a clean consolidated VM health report  
- Setup playbook installs required tools across mixed OS environments  

---

