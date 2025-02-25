# N8n Backup to GitHub Workflows

## About This Repository

This repository contains workflows and configurations for **n8n** backup processes. Specifically, it demonstrates how to **automate backups** of n8n workflows to a GitHub repository by committing any new or changed workflows into version control.

## Table of Contents
1. [Overview](#overview)  
2. [Setup](#setup)  
3. [Uploading the JSON File](#uploading-the-json-file)  
4. [Creating the Credentials](#creating-the-credentials)  
5. [Adjusting Specific Nodes](#adjusting-specific-nodes)  
6. [Credits](#credits)  
7. [Server and SSL Setup](#server-and-ssl-setup)  
8. [Potential Issues](#potential-issues)  
9. [Useful Commands](#useful-commands)  

---

## Overview

- **Goal**: Automate backing up n8n workflows into a GitHub repository.
- **Method**: Use n8n’s built-in functionality to commit any new or updated workflows to GitHub.
- **Prerequisites**: A server running n8n (with Docker), SSL enabled via reverse proxy (e.g., Traefik), and a GitHub repository to store workflow JSON files.

---

## Setup

1. **Server with SSL Authentication**  
   - You need a server that can run Docker and allow HTTPS/SSL.  
   - This example uses the **AWS free tier** (EC2) along with a **free DuckDNS subdomain**.
   - See the [Server and SSL Setup](#server-and-ssl-setup) section for detailed instructions.

2. **n8n Installation with Docker**  
   - Install n8n in a Docker container.  
   - Make sure Traefik or another reverse proxy handles SSL.  
   - Verify that your subdomain (e.g., `backupn8n.duckdns.org`) is properly pointing to your EC2 instance.

---

## Uploading the JSON File

1. **Access Your n8n Interface**  
   - Go to your subdomain address (e.g., `https://backupn8n.duckdns.org`).
   - Log in to your n8n account.

2. **Upload the Workflow**  
   - In n8n, import the JSON file named `backup-n8n-in-github...json`.  
   - This workflow is designed to back up your n8n workflows to GitHub.

---

## Creating the Credentials

1. **Create a GitHub Credential**  
   - In GitHub, go to **Settings → Developer Settings → OAuth Apps → New OAuth App**.  
   - Register a new OAuth application to get the **Client ID** and **Client Secret**.  
   - Copy these and paste them into your new GitHub credential in n8n.

2. **Create an n8n API Credential**  
   - In n8n, go to **Settings → n8n API**.  
   - Create a new API Key, then copy it into the n8n credential configuration that the workflow requires.

---

## Adjusting Specific Nodes

1. **GitHub Nodes**  
   - Fill in the fields with your GitHub **username** and **repository name**.  
   - Example: create a repository named `n8n-backups`, and ensure the node references a valid file path (e.g., `example.json`).

2. **Date Format Node**  
   - If needed, change the time zone setting in the **Format Date** node by selecting **Use Workflow Timezone**.

3. **Trigger and Scheduling**  
   - Modify the workflow’s Trigger node (e.g., Cron) to control how often backups occur.

---

## Server and SSL Setup

Below is a concise guide on how you can configure an AWS EC2 instance with Docker, Docker Compose, and a DuckDNS subdomain for SSL (using Traefik in this example).

1. **Create AWS Security Groups**  
   - In AWS, open **EC2 → Security Groups**.  
   - Add inbound rules for **HTTP (port 80)** and **HTTPS (port 443)** from `0.0.0.0/0`.

2. **Create an AWS EC2 Server (Free Tier)**  
   - Launch an **EC2 instance** (Amazon Linux, t2.micro, etc.).  
   - Use the following user-data script to install Docker, Git, Docker Compose, Node.js, and a small swap space:

     ```bash
     #!/bin/bash

     # Install Docker and Git
     sudo yum update -y
     sudo yum install git -y
     sudo yum install docker -y
     sudo usermod -a -G docker ec2-user
     sudo usermod -a -G docker ssm-user
     id ec2-user ssm-user
     sudo newgrp docker

     # Enable Docker
     sudo systemctl enable docker.service
     sudo systemctl start docker.service

     # Install Docker Compose v2
     sudo mkdir -p /usr/local/lib/docker/cli-plugins
     sudo curl -SL https://github.com/docker/compose/releases/download/v2.23.3/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose
     sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose

     # Add swap
     sudo dd if=/dev/zero of=/swapfile bs=128M count=32
     sudo chmod 600 /swapfile
     sudo mkswap /swapfile
     sudo swapon /swapfile
     echo "/swapfile swap swap defaults 0 0" | sudo tee -a /etc/fstab

     # Install Node.js and npm
     curl -fsSL https://rpm.nodesource.com/setup_21.x | sudo bash -
     sudo yum install -y nodejs
     ```

---

## Credits

- **Workflow Creation**: Credit to **Orkar**  
- **Tutorial Video**: [https://www.youtube.com/watch?v=dNuVuoPD0Jo](https://www.youtube.com/watch?v=dNuVuoPD0Jo)  
- **YouTube Channel**: [@workfloows](https://www.youtube.com/@workfloows)

- **AWS Server Setup**: Credit to **Henrylle Maia**  
- **Tutorial Video**: [https://www.youtube.com/watch?v=-gyIdyy3X0Y](https://www.youtube.com/watch?v=-gyIdyy3X0Y)  
- **YouTube Channel**: [@henryllemaia](https://www.youtube.com/@henryllemaia)

---

## Potential Issues

1. **Server IP Changes**  
   - If your EC2 instance reboots or you’re not using an Elastic IP, the public IP can change.  
   - You must update DuckDNS with the **new IP**.
   - You must update the .env with the **new IP**.

2. **Reconfiguring Subdomain**  
   - If you change subdomains, update:
     - Your **n8n credential** for the backup workflow.
     - Any **OAuth settings** in GitHub Developer → OAuth Apps.

---

## Useful Commands

From the **AWS EC2 terminal**:
```bash
sudo su ec2-user
cd /home/ec2-user/n8n

# Start/Stop containers
docker compose down
docker compose up -d

# Test if n8n is responding locally
curl -I http://127.0.0.1:5678

# Test external URL (use it in other terminal)
curl -I https://n8n-personal-assistant.duckdns.org
```
