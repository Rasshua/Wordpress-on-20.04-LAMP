# Wordpress on Ubuntu Server 20.04 with LAMP stack
Step by step installation Guide

##1. Initial Setup of Ubuntu Server##

###1-Logging in as root###

ssh root@your_server_ip

###2-Creating a new user###

adduser sammy

###3-Granting Administrative Privileges###

usermod -aG sudo sammy

###4-Setting Up a Basic Firewall###

ufw app list

Output:
Available applications:
  OpenSSH

ufw allow OpenSSH

ufw enable

ufw status

Output:
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)

###5-Enabling External Access for Your Regular User###

Via SSH





Reference: https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-on-ubuntu-20-04-with-a-lamp-stack-ru

