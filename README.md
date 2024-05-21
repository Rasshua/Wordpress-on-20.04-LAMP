# Wordpress on Ubuntu Server 20.04 with LAMP stack
Step by step installation Guide

## 1. Initial Setup of Ubuntu Server

- Logging in as root

```console
ssh root@your_server_ip
```

- Creating a new user

```console
adduser sammy
```

- Granting Administrative Privileges

```console
usermod -aG sudo sammy
```

- Setting Up a Basic Firewall

Retrieve the list of available applications:
```console
ufw app list
```
_Output:_
```
Available applications:
  OpenSSH
```
Allow port 22 for SSH:
```console
ufw allow OpenSSH
```
Enable firewall:
```console
ufw enable
```
Check the status of UFW:
```console
ufw status
```
_Output:_
```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

- Enabling External Access for Your Regular User

Via SSH

---

## 2. Install Linux, Apache, MySQL, PHP (LAMP) Stack components

### 2.1-Installing Apache and Updating the Firewall

- Update repositories:
```console
sudo apt update
```
- Install Apache2:
```console
sudo apt install apache2
```
- Retrieve available applications from UFW:
```console
sudo ufw app list
```
_Output:_
```
Available applications:
  Apache
  Apache Full
  Apache S
```
- Allow traffic on both ports 80 and 443:
```console
sudo ufw allow in "Apache Full"
```
- Verify the changes:
```console
sudo ufw status
```
_Output:_
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
```console
sudo ufw reload
```
- Try to access the website in browser:
```
http://your_server_ip
```
If everything is ok, the page like this appears:

![Apache2](https://github.com/Rasshua/Wordpress-on-20.04-LAMP/blob/main/assets/apache2.png)

#### How To Find your Server’s Public IP Address___

- via iproute tool (local IP address):
```console
ip addr show ens3 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```
- via `curl` utility (external IP address):
```console
curl http://icanhazip.com
```
---

### 2.2-Installing MySQL

- Install MySQL server:
```console
sudo apt install mysql-server
```
The following steps are intended to prevent error during mysql_secure_installation script execution:

1. Open up the MySQL prompt:
```console
sudo mysql
```
2. Change the root user’s authentication method to one that uses a password:
```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
3. Exit the MySQL prompt:
```
mysql> exit
```
- Start the interactive script for database password protection:
```
sudo mysql_secure_installation
```
_Output:_
```
VALIDATE PASSWORD PLUGIN can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No:
```
Click Y (Yes) and then select the level of password validation policy (2 - MEDIUM is recommended):
```
There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary              file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 1
```
For the rest of the questions, press Y and hit the ENTER key at each prompt.

- Test whether you’re able to log in to the MySQL console:
```
sudo mysql
```
_Output:_
```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.28-0ubuntu4 (Ubuntu)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
- Exit the MySQL console:
```
mysql> exit
```
---

### 2.3-Installing PHP

- Install the minimal necessary packages:
```
sudo apt install php libapache2-mod-php php-mysql
``` 
- Confirm your PHP version:
```
php -v
```
_Output (example):_
```
PHP 7.4.3-4ubuntu2.22 (cli) (built: May  1 2024 10:11:33) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.3-4ubuntu2.22, Copyright (c), by Zend Technologies
```

##### Changing Apache’s Directory Index (Optional)

In some cases, you’ll want to modify the way that Apache serves files when a directory is requested. By default, when the user accesses website, Apache tries to execute `index.html` file first. It's possible to modify default directory index so Apache will serve `index.php` first.

1. Open the configuration file in the text editor:
```
sudo nano /etc/apache2/mods-enabled/dir.conf
```
_File content (`index.html` is first in queue):
```
<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
</IfModule>
```
2. Change the line so that the index is first:
```
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
3. Save changes and than restart the Apache server:
```
sudo systemctl restart apache2
```
---

### 2.4-Creating a Virtual Host for website

When using the Apache web server, you can create virtual hosts to encapsulate configuration details and host more than one domain from a single server. In this example, we’ll set up a domain called `your_domain`, but you should replace this with your own domain name.

- Create the directory for your_domain
```
sudo mkdir /var/www/your_domain
```
- Assign ownership of the directory with the $USER environment variable, which will reference your current system user:
```
sudo chown -R $USER:$USER /var/www/your_domain
```
- Open a new configuration file in Apache’s `sites-available` directory:
```
sudo nano /etc/apache2/sites-available/your_domain.conf
```
- Add in the following bare-bones configuration with your own domain name:
```
<VirtualHost *:80>
    ServerName your_domain
    ServerAlias www.your_domain
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/your_domain
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Save and close the file when you’re done.

- Use `a2ensite` to enable the new virtual host:
```
sudo a2ensite your_domain
```
- Disable the default website that comes installed with Apache (this is required if you’re not using a custom domain name, because in this case Apache’s default configuration would override your virtual host):
```
sudo a2dissite 000-default
```
- Make sure your configuration file doesn’t contain syntax errors:
```
sudo apache2ctl configtest
```
If `Syntax OK` appears in output, everything is OK.

- Reload Apache so these changes take effect:
```
sudo systemctl reload apache2
```
---

### 2.4-Testing PHP processing on the website

- Create a new file named info.php inside your custom web root folder:
```
nano /var/www/your_domain/info.php
```
- Add the following text, which is valid PHP code, inside the blank file:
```php
<?php
phpinfo();
?>
```
Save and close the file.

- Go to your web browser and access your server’s domain name or IP address, followed by the script name:
```
http://server_domain_or_IP/info.php
```
If everything is ok, default PHP page appears:

![PHP](https://github.com/Rasshua/Wordpress-on-20.04-LAMP/blob/main/assets/php.png)




Reference: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-on-ubuntu-20-04-with-a-lamp-stack-ru)

