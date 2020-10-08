+++
title = "How to harden Debian Buster"
date = "2020-10-08"
author = "Selwyn"
cover = ""
draft = "false"
description = "This guide will show you how to harden your Debian Buster server by setting up UFW (Uncomplicated Firewall), Fail2Ban and modifying SSH configuration." 
+++

# How to harden Debian Buster
## Update & upgrade
```bash
apt-get update -y && apt-get upgrade -y
```

## Install necessary packages
```bash
apt-get install -y vim git ufw fail2ban sudo
```
## User Setup
### Add new user 
This step can be skipped if you allready have a user with sudo rights that is not root.
```bash
useradd -m -s /bin/bash your_username
```
### Generate SSH Key on your local machine
On your local machine generate a SSH Key pair. Then copy the content of your public key. (`id_rsa.pub`)
````bash
ssh-keygen -t rsa -b 4096 -C "yourname@youremail"
`` 
### SSH Keys
Paste public key to `~/.ssh/authorized_keys`
```bash
mkdir ~/.ssh
touch ~/.ssh/authorized_keys
chmod 755 ~/.ssh
chmod 644 ~/authorized_keys
```
### Add user to sudo group
```bash
usermod -aG sudo your_username
```
## SSH Configuration
### Disable root and password login
Edit `/etc/sshd/sshd_config` with your preferred text editor (e.g. vim or nano)
```bash
vim /etc/ssh/sshd_config
```
Search for and change following settings:
```bash
PermitRootLogin no
ChallengeResponseAuthentication no
PasswordAuthentication no
UsePAM no
```
Reload SSH service:
```bash
systemctl reload ssh
```
## Configure UFW firewall
### Enable and start UFW
```bash
systemctl enable ufw
systemctl start ufw
```
### Set default UFW rules
This will deny all incoming and allow all outgoing connections:
```bash
ufw default deny incoming
ufw default allow incoming
```
Make sure you allow ssh connections to be able to login:
```bash
ufw allow ssh
```
Optionally allow https if your running web applications. You can also allow http although I prefer for better security using only https.
```bash
ufw allow https
# ufw allow http
```
Enable rules and check status
```bash
ufw enable
ufw status
```
You should get following status output:
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
This is my basic fail2ban configuration.
Create local jail configuration and paste the below configuration.
```bash
vim /etc/fail2ban/jail.local
```
```
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxtretry = 5
```
Restart fail2ban service:
```bash
systemctl restart fail2ban
```
