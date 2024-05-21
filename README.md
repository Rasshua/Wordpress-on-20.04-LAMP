# Wordpress on Ubuntu Server 20.04 with LAMP stack
Step by step installation Guide

## 1. Initial Setup of Ubuntu Server

### 1.1-Logging in as root

```
ssh root@your_server_ip
```

### 1.2-Creating a new user

```
adduser sammy
```

### 1.3-Granting Administrative Privileges

```
usermod -aG sudo sammy
```

### 1.4-Setting Up a Basic Firewall

Retrieve the list of available applications:
```
ufw app list
```
Output:
```
Available applications:
  OpenSSH
```
Allow port 22 for SSH:
```
ufw allow OpenSSH
```
Enable firewall:
```
ufw enable
```
Check the status of UFW:
```
ufw status
```
Output:
```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

### 1.5-Enabling External Access for Your Regular User

Via SSH

## 2. Install Linux, Apache, MySQL, PHP (LAMP)

### 2.1-Installing Apache and Updating the Firewall

- Update repositories:
```
sudo apt update
```
- Install Apache2:
```
sudo apt install apache2
```
- Retrieve available applications from UFW:
```
sudo ufw app list
```
Output:
```
Available applications:
  Apache
  Apache Full
  Apache S
```
- Allow traffic on both ports 80 and 443:
```
sudo ufw allow in "Apache Full"
```
- Verify the changes:
```
sudo ufw status
```
Output:
```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Apache Full                ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Apache Full (v6)           ALLOW       Anywhere (v6)
```
- Restart UFW with new rules:
```
sudo ufw reload
```
- Try to access the website in browser:
```
http://your_server_ip
```
If everything is ok, the page like this appears:





Reference: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-on-ubuntu-20-04-with-a-lamp-stack-ru)

