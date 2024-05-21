# WordPress on Ubuntu Server 20.04 with LAMP stack
Step by step installation Guide

## 1. Initial Setup of Ubuntu Server

1. Logging in as root
```console
ssh root@your_server_ip
```
2. Creating a new user
```console
adduser sammy
```
3. Granting Administrative Privileges
```console
usermod -aG sudo sammy
```
4. Setting Up a Basic Firewall

- Retrieve the list of available applications:
```console
ufw app list
```
_Output:_
```
Available applications:
  OpenSSH
```
- Allow port 22 for SSH:
```console
ufw allow OpenSSH
```
- Enable firewall:
```console
ufw enable
```
- Check the status of UFW:
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

5. Enabling External Access for Your Regular User

Via SSH

---

## 2. Install LAMP Stack components - Apache, MySQL and PHP

### 2.1-Installing Apache and Updating the Firewall

1. Update repositories:
```console
sudo apt update
```
2. Install Apache2:
```console
sudo apt install apache2
```
3. Retrieve available applications from UFW:
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
4. Allow traffic on both ports 80 and 443:
```console
sudo ufw allow in "Apache Full"
```
5. Verify the changes:
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
6. Restart UFW with new rules:
```console
sudo ufw reload
```
7. Try to access the website in browser:
```
http://your_server_ip
```
If everything is ok, the page like this appears:

![Apache2](https://github.com/Rasshua/Wordpress-on-20.04-LAMP/blob/main/assets/apache2.png)

#### ==== How To Find your Server’s Public IP Address ====

- via `iproute` tool (local IP address):
```console
ip addr show ens3 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```
- via `curl` utility (external IP address):
```console
curl http://icanhazip.com
```
---

### 2.2-Installing MySQL

1. Install MySQL server:
```console
sudo apt install mysql-server
```
The following steps are intended to prevent error during mysql_secure_installation script execution:

- Open up the MySQL prompt:
```console
sudo mysql
```
- Change the root user’s authentication method to one that uses a password:
```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
- Exit the MySQL prompt:
```sql
exit
```
2. Start the interactive script for database password protection:
```console
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

3. Test whether you’re able to log in to the MySQL console:
```console
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
4. Exit the MySQL console:
```sql
exit
```
---

### 2.3-Installing PHP

1. Install the minimal necessary packages:
```console
sudo apt install php libapache2-mod-php php-mysql
``` 
2. Confirm your PHP version:
```console
php -v
```
_Output (example):_
```
PHP 7.4.3-4ubuntu2.22 (cli) (built: May  1 2024 10:11:33) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.3-4ubuntu2.22, Copyright (c), by Zend Technologies
```

#### ==== Changing Apache’s Directory Index (Optional) ====

In some cases, you’ll want to modify the way that Apache serves files when a directory is requested. By default, when the user accesses website, Apache tries to execute `index.html` file first. It's possible to modify default directory index so Apache will serve `index.php` first.

- Open the configuration file in the text editor:
```console
sudo nano /etc/apache2/mods-enabled/dir.conf
```
_File content (`index.html` is first in queue):
```
<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
</IfModule>
```
- Change the line so that the index is first:
```
<IfModule mod_dir.c>
    DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
- Save changes and than restart the Apache server:
```console
sudo systemctl restart apache2
```
---

### 2.4-Creating a Virtual Host for website

When using the Apache web server, you can create virtual hosts to encapsulate configuration details and host more than one domain from a single server. In this example, we’ll set up a domain called `your_domain`, but you should replace this with your own domain name.

1. Create the directory for your_domain
```console
sudo mkdir /var/www/your_domain
```
2. Assign ownership of the directory with the $USER environment variable, which will reference your current system user:
```console
sudo chown -R $USER:$USER /var/www/your_domain
```
3. Open a new configuration file in Apache’s `sites-available` directory:
```console
sudo nano /etc/apache2/sites-available/your_domain.conf
```
4. Add in the following bare-bones configuration with your own domain name:
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

5. Use `a2ensite` to enable the new virtual host:
```console
sudo a2ensite your_domain
```
6. Disable the default website that comes installed with Apache (this is required if you’re not using a custom domain name, because in this case Apache’s default configuration would override your virtual host):
```console
sudo a2dissite 000-default
```
7. Make sure your configuration file doesn’t contain syntax errors:
```console
sudo apache2ctl configtest
```
If `Syntax OK` appears in output, everything is OK.

8. Reload Apache so these changes take effect:
```console
sudo systemctl reload apache2
```
---

### 2.4-Testing PHP processing on the website

1. Create a new file named info.php inside your custom web root folder:
```console
nano /var/www/your_domain/info.php
```
2. Add the following text, which is valid PHP code, inside the blank file:
```php
<?php
phpinfo();
?>
```
Save and close the file.

3. Go to your web browser and access your server’s domain name or IP address, followed by the script name:
```
http://server_domain_or_IP/info.php
```
If everything is ok, default PHP page appears:

![PHP](https://github.com/Rasshua/Wordpress-on-20.04-LAMP/blob/main/assets/php.png)

4. After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains sensitive information:
```console
sudo rm /var/www/your_domain/info.php
```
---

## 3. Install WordPress

### 3.1-Creating a MySQL Database and User for WordPress

1. Login to the root account in MySQL as regular Linux user:

```console
mysql -u root -p
```
2. Create database for WordPress (`wordpress` in example):
```sql
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```
3. Create user for WordPress (`wordpressuser` in example, `password` must be substituted with particular value):
```sql
CREATE USER 'wordpressuser'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
```
4. Grant all access to database for WordPress user:
```sql
GRANT ALL ON wordpress.* TO 'wordpressuser'@'%';
```
5. Refresh privileges:
```sql
FLUSH PRIVILEGES;
```
6. Exit MySQL:
```sql
exit
```
---

### 3.2-Install additional extensions for PHP

1. Update repositories:
```console
sudo apt update
```
2. Install PHP extensions:
```console
sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
```
3. Restart Apache server:
```console
sudo systemctl restart apache2
```
---

### 3.3-Setting up Apache configuration to gain access to .htaccess override and rewrite

We will use `/etc/apache2/sites-available/your_domain.conf` as an example and you need to replace the path to your configuration file in the appropriate location. We will also use `/var/www/your_domain` as the root directory for our WordPress installation. You must use the root web site specified in your own configuration.

1. Open the configuration file of Apache server for yout website:
```console
sudo nano /etc/apache2/sites-available/your_domain.conf
```
2. Add the following text block inside the `VirtualHost` block in your configuration file, making sure you are using the correct web root directory:
```
<Directory /var/www/your_domain/>
	AllowOverride All
</Directory>
```
Save the file and exit editor.

3. Activate the rewrite module to use permalinks in WordPress:
```console
sudo a2enmod rewrite
```
This will give you easier-to-read permalinks for your posts, as shown:
```
http://example.com/2012/post-name/
http://example.com/2012/12/30/post-name
```
4. Check syntax:
```console
sudo apache2ctl configtest
```
5. Restart Apache server:
```console
sudo systemctl restart apache2
```
---

### 3.4-Downloading WordPress

1. Jump to `tmp` directory and download the compresser release:
```console
cd /tmp
curl -O https://wordpress.org/latest.tar.gz
```
2. Unpack the compressed file to create directory structure of WordPress:
```console
tar xzvf latest.tar.gz
```
3. Create empty `.htaccess` file in directory structure:
```console
touch /tmp/wordpress/.htaccess
```
4. Rename the configuration file in directory structure:
```console
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
```
5. Create an `upgrade` directory so that WordPress doesn't have permission issues when trying to do this on its own after upgrading the software:
```console
mkdir /tmp/wordpress/wp-content/upgrade
```
6. Copy all directory structure and content to the root directory of our website:
```console
sudo cp -a /tmp/wordpress/. /var/www/your_domain
```
---

### 3.5-Setting up a WordPress directory

1. Setting up smart ownership and permissions:
```console
sudo chown -R www-data:www-data /var/www/wordpress
sudo find /var/www/wordpress/ -type d -exec chmod 750 {} \;
sudo find /var/www/wordpress/ -type f -exec chmod 640 {} \;
```
2. Setting up the WordPress configuration file:

- Generate secure values ​​from WordPress secret key generator
```console
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```
_Output example:_
```
define('AUTH_KEY',         '1jl/vqfs<XhdXoAPz9 DO NOT COPY THESE VALUES c_j{iwqD^<+c9.k<J@4H');
define('SECURE_AUTH_KEY',  'E2N-h2]Dcvp+aS/p7X DO NOT COPY THESE VALUES {Ka(f;rv?Pxf})CgLi-3');
define('LOGGED_IN_KEY',    'W(50,{W^,OPB%PB<JF DO NOT COPY THESE VALUES 2;y&,2m%3]R6DUth[;88');
define('NONCE_KEY',        'll,4UC)7ua+8<!4VM+ DO NOT COPY THESE VALUES #`DXF+[$atzM7 o^-C7g');
define('AUTH_SALT',        'koMrurzOA+|L_lG}kf DO NOT COPY THESE VALUES  07VC*Lj*lD&?3w!BT#-');
define('SECURE_AUTH_SALT', 'p32*p,]z%LZ+pAu:VY DO NOT COPY THESE VALUES C-?y+K0DK_+F|0h{!_xY');
define('LOGGED_IN_SALT',   'i^/G2W7!-1H2OQ+t$3 DO NOT COPY THESE VALUES t6**bRVFSD[Hi])-qS`|');
define('NONCE_SALT',       'Q6]U:K?j4L%Z]}h^q7 DO NOT COPY THESE VALUES 1% ^qUswWgn+6&xqHN&%');
```
Copy the string block to put it into configuration file.

- Open the WordPress configuration file:
```console
sudo nano /var/www/your_domain/wp-config.php
```
- Find the section with dummy values for these settings:
```
.  .  .

define('AUTH_KEY',         'put your unique phrase here');
define('SECURE_AUTH_KEY',  'put your unique phrase here');
define('LOGGED_IN_KEY',    'put your unique phrase here');
define('NONCE_KEY',        'put your unique phrase here');
define('AUTH_SALT',        'put your unique phrase here');
define('SECURE_AUTH_SALT', 'put your unique phrase here');
define('LOGGED_IN_SALT',   'put your unique phrase here');
define('NONCE_SALT',       'put your unique phrase here');

.  .  .
```

- Remove strings with dummy values and substitute them by string block from WordPress secret key generator:
```
.  .  .

define('AUTH_KEY',         '1jl/vqfs<XhdXoAPz9 DO NOT COPY THESE VALUES c_j{iwqD^<+c9.k<J@4H');
define('SECURE_AUTH_KEY',  'E2N-h2]Dcvp+aS/p7X DO NOT COPY THESE VALUES {Ka(f;rv?Pxf})CgLi-3');
define('LOGGED_IN_KEY',    'W(50,{W^,OPB%PB<JF DO NOT COPY THESE VALUES 2;y&,2m%3]R6DUth[;88');
define('NONCE_KEY',        'll,4UC)7ua+8<!4VM+ DO NOT COPY THESE VALUES #`DXF+[$atzM7 o^-C7g');
define('AUTH_SALT',        'koMrurzOA+|L_lG}kf DO NOT COPY THESE VALUES  07VC*Lj*lD&?3w!BT#-');
define('SECURE_AUTH_SALT', 'p32*p,]z%LZ+pAu:VY DO NOT COPY THESE VALUES C-?y+K0DK_+F|0h{!_xY');
define('LOGGED_IN_SALT',   'i^/G2W7!-1H2OQ+t$3 DO NOT COPY THESE VALUES t6**bRVFSD[Hi])-qS`|');
define('NONCE_SALT',       'Q6]U:K?j4L%Z]}h^q7 DO NOT COPY THESE VALUES 1% ^qUswWgn+6&xqHN&%');

.  .  .
```

- You need to change some database connection parameters at the beginning of the file. You need to change the database name (`wordpress`), database user (`wordpressuser`) and corresponding `password` that we have previously configured in MySQL. You also need to define the method that WordPress should use to write data to the file system:
```
. . .

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wordpressuser' );

/** MySQL database password */
define( 'DB_PASSWORD', 'password' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );


. . .

define('FS_METHOD', 'direct');
```

- Save and close the configuration file.
---

### 3.6-Completing installation via the web interface



---
Reference: [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-on-ubuntu-20-04-with-a-lamp-stack-ru)

