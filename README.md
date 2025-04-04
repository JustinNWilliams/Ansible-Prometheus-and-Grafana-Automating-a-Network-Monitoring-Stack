
# Automating a Network Monitoring Stack with Ansible

## Introduction

In this project, I used **Ansible** to automate the deployment of a network monitoring stack on an **Ubuntu 24.04** VM running in **Oracle VM VirtualBox**. I set up **Prometheus** for monitoring system metrics and **Grafana** for visualization, complete with basic alerting, showcasing my skills in IT automation, network monitoring, and observability.

---

## What is Ansible?

![image](https://github.com/user-attachments/assets/cd4a1546-73a9-4e54-8896-c3f06f96b41b)

**Ansible** is an open-source automation tool that simplifies IT tasks like configuration management, application deployment, and network automation. It enables users to:
- Automate repetitive tasks with playbooks.
- Manage infrastructure as code.
- Scale across multiple systems with an agentless architecture.

---

## Project Overview

This project demonstrates:
- Installing and configuring **Ansible** on Ubuntu 24.04.
- Writing an Ansible playbook to deploy **Prometheus** and **Grafana**.
- Configuring Prometheus to scrape system metrics.
- Setting up Grafana to visualize metrics and connect to Prometheus.
- Documenting the process for a professional portfolio.

---

## Prerequisites

To complete this project, I needed:
- An **Ubuntu 24.04** VM running in **Oracle VM VirtualBox**.
- Basic familiarity with Linux commands and YAML syntax.
- Internet access for downloading packages.
- Port forwarding configured in VirtualBox (SSH: 2222 → 22, Grafana: 3000 → 3000, Prometheus: 9090 → 9090).

---

## Steps to Complete the Project

### 1. Set Up the Ansible Environment

I began by installing Ansible on my Ubuntu VM to serve as the control node for automation.

Updated the package list and installed Ansible:
```bash
sudo apt update
sudo apt install ansible -y
ansible --version
```

Created a project directory:
```bash
mkdir ansible-monitoring && cd ansible-monitoring
```

![Screenshot 2025-04-04 024420](https://github.com/user-attachments/assets/2d64c73d-93a6-4c31-945c-94537732d010)


---

### 2. Create an Ansible Inventory and Playbook Structure

I set up the inventory and playbook to define the target system (localhost) and automation tasks.

Created an inventory file:
```bash
touch inventory.yml
```

Contents of `inventory.yml`:
```yaml
all:
  hosts:
    localhost:
      ansible_connection: local
      ansible_python_interpreter: /usr/bin/python3
```

Created a playbook file:
```bash
touch monitoring.yml
```

![Screenshot 2025-04-04 024625](https://github.com/user-attachments/assets/be51de56-6941-4c7a-8044-f9e403813e22)

---

### 3. Write an Ansible Playbook to Install Prometheus and Grafana

I wrote a playbook to automate the installation and configuration of Prometheus and Grafana.

Contents of `monitoring.yml`:
```yaml
name: Deploy Monitoring Stack
hosts: localhost
become: yes
tasks:
  - name: Update apt cache
    apt:
      update_cache: yes

  - name: Install prerequisites
    apt:
      name: "{{ item }}"
      state: present
    loop:
      - curl
      - wget

  - name: Add Prometheus user
    user:
      name: prometheus
      system: yes
      create_home: no

  - name: Download Prometheus
    get_url:
      url: https://github.com/prometheus/prometheus/releases/download/v2.51.0/prometheus-2.51.0.linux-amd64.tar.gz
      dest: /tmp/prometheus.tar.gz

  - name: Extract Prometheus
    unarchive:
      src: /tmp/prometheus.tar.gz
      dest: /usr/local/bin/
      remote_src: yes
      creates: /usr/local/bin/prometheus-2.51.0.linux-amd64/prometheus

  - name: Install Grafana APT key
    apt_key:
      url: https://packages.grafana.com/gpg.key
      state: present

  - name: Add Grafana repository
    apt_repository:
      repo: deb https://packages.grafana.com/oss/deb stable main
      state: present

  - name: Install Grafana
    apt:
      name: grafana
      state: present
```

💡 This playbook automates the setup of Prometheus and Grafana, ensuring consistency and repeatability.

![Screenshot 2025-04-04 025446](https://github.com/user-attachments/assets/c97cde7d-b0df-4cd1-926c-66652852eec3)

---

### 4. Run the Playbook and Verify Installation

I executed the playbook and verified that Prometheus and Grafana were running.

Ran the playbook:
```bash
ansible-playbook -i inventory.yml monitoring.yml
```

![Screenshot 2025-04-04 030144](https://github.com/user-attachments/assets/033c1494-12fe-4d32-ac99-30e058d10641)

Started Prometheus manually:
```bash
sudo /usr/local/bin/prometheus-2.51.0.linux-amd64/prometheus --config.file=/usr/local/bin/prometheus-2.51.0.linux-amd64/prometheus.yml &
```

Started Grafana:
```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

Accessed Grafana at [http://localhost:3000](http://localhost:3000) and Prometheus at [http://localhost:9090](http://localhost:9090) via my host browser using port forwarding.

![Screenshot 2025-04-04 031025](https://github.com/user-attachments/assets/532fa9f8-3098-4ddc-9ac8-823517696eb6)
![Screenshot 2025-04-04 031018](https://github.com/user-attachments/assets/5f7bdf2c-3e52-4d7f-ae04-40d6d469deec)

---

### 5. Configure Basic Monitoring and Alerting

I configured Prometheus to scrape metrics and connected it to Grafana for visualization.

Created `prometheus-config.yml`:
```bash
touch prometheus-config.yml
```

Contents:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'local_system'
    static_configs:
      - targets: ['localhost:9090']

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']
```

Restarted Prometheus with the new config:
```bash
pkill prometheus
sudo /usr/local/bin/prometheus-2.51.0.linux-amd64/prometheus --config.file=prometheus-config.yml > prometheus.log 2>&1 &
```

Added Prometheus as a data source in Grafana at `http://localhost:9090` and tested the connection.

![Screenshot 2025-04-04 033751](https://github.com/user-attachments/assets/c8c5923e-2a84-4521-a6f5-a39e5322e9b8)

---

## Key Takeaways

- I learned that Ansible is a powerful tool for automating IT tasks, making deployments consistent and efficient.
- This project helped me understand how to set up a monitoring stack with Prometheus and Grafana, key tools for observability.
- I gained hands-on experience in troubleshooting SSH, process management, and network connectivity issues in a Linux environment.

---

## Resources

- [Ansible Documentation](https://docs.ansible.com/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Ubuntu 24.04 Documentation](https://ubuntu.com/)
- [VirtualBox Networking Guide](https://www.virtualbox.org/manual/ch06.html)
