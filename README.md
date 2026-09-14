# 📘  Ansible EC2 Automation & VM Metrics**

## 📌 Overview

This project uses **Ansible** to automate package installation (Docker, Maven), create directories, and collect **system metrics** (CPU, Memory, Disk) from multiple EC2 instances.  
It also uses the **AWS EC2 Dynamic Inventory Plugin** to automatically fetch host IP addresses based on EC2 tags.

You can run:

- A **setup playbook** → installs Docker, Maven, net-tools, creates directories  
- A **metrics playbook** → gathers CPU, memory, disk usage and emails a consolidated report  
- A **ping test** → verifies connectivity to all EC2 hosts

---

## 📁 Project Structure

```
inventory/
│
├── aws_ec2.yaml          # Dynamic inventory configuration
├── group_vars/
│     ├── os_ubuntu.yaml  # Ubuntu-specific variables
│     └── os_amazon.yaml  # Amazon Linux-specific variables
│
├── playbook-setup.yaml   # Installs packages (Docker, Maven)
└── playbook-metrics.yaml # Collects VM metrics & sends report
```

---

## 🌐 Dynamic Inventory (AWS EC2)

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

