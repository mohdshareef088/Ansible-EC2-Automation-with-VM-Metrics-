# 📘  **Ansible EC2 Automation & VM Metrics**

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
- Tagging the hosts machines OS tagging (`os=ubuntu`, `os=amazon`)#tag.sh
Name=Environment, Values=dev
Name=os,Values=ubuntu #if the host is ubuntu
Name=os,Values=amazon #if the host is redhat distro
  
<img width="621" height="77" alt="image" src="https://github.com/user-attachments/assets/b56cefdd-c4f0-41c4-8bac-9e5649e30787" />

- Generate ssh-keygen for the master node and copy the master.pem file and injecting ssh public key into hosts #copy_pub.sh
- Run the Dynamic inventory automatically updates when EC2 instances to show discovered IP addresses #ansible-inventory -i inventory/aws_ec2.yaml --graph 
<img width="704" height="418" alt="image" src="https://github.com/user-attachments/assets/9234644a-e600-4ba4-9220-98c3da251e24" />

## 🧪 Testing Connectivity
- pinging the ping pong output to the remote hosts, categorizing Ubuntu and Amazon hosts
- @os_ubuntu
  └── ec2-13-203-160-135.ap-south-1.compute.amazonaws.com
- @os_amazon
  └── ec2-13-127-79-63.ap-south-1.compute.amazonaws.com
<img width="1587" height="525" alt="image" src="https://github.com/user-attachments/assets/b9624be4-947d-49f8-96ed-513870590fcf" />


## 🚀 Running the Playbooks

### 1️⃣ Run setup (install Docker, Maven)

```
- playbook installs required tools across mixed OS environments 
  #ansible-playbook -i inventory/aws_ec2.yaml playbook.yaml
```
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
```
`playbook.yaml`:

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
<img width="1222" height="765" alt="image" src="https://github.com/user-attachments/assets/09cd30df-1846-42c7-ae31-3b59d6f95774" />

<img width="1189" height="763" alt="image" src="https://github.com/user-attachments/assets/57a791ff-8b2f-4f13-906b-2eb49893d2ff" />

<img width="826" height="449" alt="image" src="https://github.com/user-attachments/assets/c7e20b65-97aa-45d2-afb3-4fe3f6f1afbe" />

<img width="1285" height="172" alt="image" src="https://github.com/user-attachments/assets/6884c014-58d4-44de-afef-513835285123" />

---
---
### 2️⃣ Run metrics collection

```
#ansible-playbook -i inventory/aws_ec2.yaml collect_metrics.yaml
```
- Metrics playbook gives a clean consolidated VM health report
- adding smtp and email credentials to all.yaml to group_vars  
- the `aws_ec2.yaml` automatically discovers EC2 instances using tags
  
<img width="1580" height="823" alt="image" src="https://github.com/user-attachments/assets/7539826f-dbbe-484b-b21a-cbebb2198863" />

<img width="1612" height="1552" alt="image" src="https://github.com/user-attachments/assets/f8c7190c-e557-4a40-b9cf-0b487d19f5a8" />

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
3 VMs | Avg CPU: 0.5% | Avg Mem: 35.68% | Avg Disk: 21.67%
```




