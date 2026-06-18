# 🚀 AWS SFTP Server Setup (Ubuntu 22.04)

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![SSH](https://img.shields.io/badge/Protocol-SFTP-blue?style=for-the-badge)

> A complete, step-by-step manual setup guide to deploy a secure, production-ready SFTP server on an AWS EC2 instance. This setup uses a Chroot Jail to ensure users cannot escape their designated folders.

---

## 📖 Table of Contents
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation Commands](#installation-commands)
- [Connection Details](#connection-details)
- [Project Structure](#project-structure)
- [How to Use the Scripts](#how-to-use-the-scripts)
- [Troubleshooting](#troubleshooting)
- [Author](#author)

---

## ✨ Features
- 🔒 **Chroot Jail Security:** Users are locked to their home directory for maximum safety.
- ⚡ **Static Elastic IP:** Persistent public IP address that doesn't change on reboot.
- 📁 **Pre-configured User:** Ready-to-use `SFTP` user with password `xyz@123`.
- 🛡️ **Firewall Protection:** UFW configured to only allow port 22.
- 🐧 **Cross-Platform:** Compatible with WinSCP (Windows), FileZilla, and Linux/Mac terminals.

---

## 📋 Prerequisites
Before running the installation, ensure you have:
- An AWS EC2 Instance running **Ubuntu 22.04**.
- An **Elastic IP** associated with your instance (for a static public IP).
- **Port 22 (SSH)** open in your AWS Security Group.
- Root or sudo access to the server.

---

## 🛠️ Installation Commands

You can run this script directly on your AWS server to set up everything in one go:

```bash
#!/bin/bash
# Run this as a root user on your EC2 instance

sudo apt update && sudo apt upgrade -y
sudo apt install openssh-server -y
sudo systemctl start ssh && sudo systemctl enable ssh

# Create user 'SFTP' and set password 'xyz@123'
sudo useradd -m -s /usr/sbin/nologin SFTP
echo "SFTP:xyz@123" | sudo chpasswd

# Create secure directory structure
sudo mkdir -p /sftp/SFTP/data
sudo chown root:root /sftp && sudo chmod 755 /sftp
sudo chown root:root /sftp/SFTP && sudo chmod 755 /sftp/SFTP
sudo chown SFTP:SFTP /sftp/SFTP/data && sudo chmod 755 /sftp/SFTP/data

# Configure SSH for SFTP only (Chroot Jail)
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
sudo sed -i 's/Subsystem sftp \/usr\/lib\/openssh\/sftp-server/Subsystem sftp internal-sftp/' /etc/ssh/sshd_config
echo "
Match User SFTP
    ChrootDirectory /sftp/SFTP
    ForceCommand internal-sftp
    X11Forwarding no
    AllowTcpForwarding no
    PermitTTY no
    PasswordAuthentication yes
" | sudo tee -a /etc/ssh/sshd_config

sudo systemctl restart ssh

# Configure firewall
sudo ufw allow 22/tcp && sudo ufw enable

echo "✅ SFTP Server Setup Complete!"
