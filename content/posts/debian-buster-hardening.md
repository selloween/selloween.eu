---
author: "Selwyn Rogers"
title: "How to harden Debian Buster"
date: "2020-10-08"
description: "In this guide, I will show you how to enhance the security of your Debian Buster server by setting up UFW (Uncomplicated Firewall), Fail2Ban, and modifying the SSH configuration."
tags: ["linux","security","debian"]
ShowToc: false
ShowBreadCrumbs: false
---

# How to harden Debian Buster

## Update system
To install the necessary dependencies, you can use apt. Run the following command to update the package list and upgrade the system:
```bash
apt-get update -y && apt-get upgrade -y
```

Then, install the required packages:
```bash
apt-get install -y git ufw fail2ban sudo
```

## Add a new user 
You can skip this step if you already have a user with sudo privileges that is not root.
```bash
useradd -m -s /bin/bash your_username
```
### SSH Keys
To begin, generate an SSH Key pair on your local machine. Once generated, copy the contents of your public key. (`id_rsa.pub`)
```bash
ssh-keygen -t rsa -b 4096 -C "yourname@youremail"
```

Paste your public key into the `~/.ssh/authorized_keys` file. This will allow you to use SSH to log in to the server securely.
```bash
mkdir ~/.ssh
touch ~/.ssh/authorized_keys
chmod 755 ~/.ssh
chmod 644 ~/authorized_keys
```
### Sudo privileges
To give a user sudo privileges, you can add them to the sudo group. To do so, run the following command:
```bash
sudo usermod -aG sudo <username>
```
Replace `<username>` with the actual name of the user you want to give sudo privileges to. After running this command, the user will have sudo privileges and be able to perform administrative tasks on the server.

## SSH Server Configuration

To enhance the security of your SSH server, you should disable root and password login. To do so, follow these steps:

1. Edit the `/etc/ssh/sshd_config` file with your preferred text editor. For example, you can use `vim` or `nano`:
```bash
sudo nano /etc/ssh/sshd_config
```
2. Search for and change the following settings:
```perl
PermitRootLogin no
ChallengeResponseAuthentication no
PasswordAuthentication no
UsePAM no
```
These settings disable root login, challenge-response authentication, password authentication, and PAM authentication.
3. Save the changes to the `/etc/ssh/sshd_config` file and exit the editor.
4. Reload the SSH service to apply the changes:
```bash
sudo systemctl reload ssh
```

## Configure UFW firewall
To configure the UFW firewall, follow these steps:

1. Enable and start the UFW service:
```bash
sudo systemctl enable ufw
sudo systemctl start ufw
```
2. Set the default UFW rules to deny all incoming connections and allow all outgoing connections:
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
3. Allow SSH connections to the server to be able to log in:
```bash
sudo ufw allow ssh
```
4. Optionally, allow HTTPS if you're running web applications. You can also allow HTTP, although for better security, using only HTTPS is recommended:
```bash
sudo ufw allow https
# sudo ufw allow http
```
5. Enable the rules and check the status:
```bash
sudo ufw enable
sudo ufw status
```
You should see the following status output:
```bash
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
22/tcp (v6)                ALLOW       Anywhere (v6)             
443/tcp (v6)               ALLOW       Anywhere (v6)             
```
## Configuring fail2Ban
To configure Fail2Ban, follow these steps:

1. Create a local jail configuration file and paste the following configuration:
```bash
sudo nano /etc/fail2ban/jail.local
```java
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
```
This configuration will monitor the /var/log/auth.log file for failed SSH login attempts and block the IP addresses that exceed the maxretry limit of 5.
2. Restart the Fail2Ban service:
```
sudo systemctl restart fail2ban
```

After completing these steps, Fail2Ban will monitor your server's logs for failed SSH login attempts and block IP addresses that exceed the maxretry limit.

### Additonal security measures
In addition to the steps outlined in this guide, here are some other measures you can take to further harden your Debian server:

1. Keep your server up-to-date with security patches and updates by running regular system updates:
```bash
sudo apt update && sudo apt upgrade
```
2. Use strong and unique passwords for all user accounts, especially for the root and sudo users.
3. Limit user access and permissions to only what is necessary for them to perform their tasks.
4. Disable unused services and daemons on your server to reduce the attack surface.
5. Use a secure method to transfer files to and from your server, such as SFTP or SCP.
6. Enable SELinux or AppArmor to provide additional security controls.
7. Install and configure a host-based intrusion detection system (HIDS) such as OSSEC or Tripwire.

By implementing these additional measures, you can further enhance the security of your Debian server and protect it against a wide range of threats.