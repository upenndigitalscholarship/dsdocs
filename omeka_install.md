# Step 1 - Install Requirements

These instructions are for setting up Omeka 2.5.1 on Ubuntu 16.04.

Before you begin with this guide, you should have a separate, non-root user account with sudo privileges set up on your server. Use this account throughout these instructions.

## Install Apache2

```
$ sudo apt-get update
$ sudo apt-get apache2
```
Since you are using a sudo command, these operations get executed with root privileges. It will ask you for your password the first time you use `sudo`.

Press `Y` to agree to all install questions.

Set global ServerName:

```
$ sudo nano /etc/apache2/apache2.conf
```

This opens the global Apache configuration file in the nano text editor. Key down to the bottom of the file (`Ctl+V` will page down, `Ctl+Y` will page up, `Alt+/` will take you to the bottom of the file, `Alt+\` will take you to the top of the file), and add:

#### *apache2.conf*
```
ServerName server_domain_or_IP
```

(fill in the `server_domain_or_IP` with the domain name or IP of your server)

Press `Ctl+O` and `Enter` to save, then `Ctl+X` to exit.

Check for syntax errors:

```
$ sudo apache2ctl configtest
```

Should return: `Syntax OK`

Restart Apache to apply your changes:

```
$ sudo systemctl restart apache2
```

Check that your firewall is configured properly:

```
$ sudo ufw app list
```

Should return:

#### *Output*
```
Available applications:
    Apache
    Apache Full
    Apache Secure
    OpenSSH
```

```
$ sudo ufw app info "Apache Full"
```

Should return:

#### *Output*
```
Profile: Apache Full
Title: Web Server (HTTP,HTTPS)
Description: Apache v2 is the next generation of the omnipresent Apache web
server.

Ports:
  80,443/tcp
```

Allow incoming traffic:

```
$ sudo ufw allow in "Apache Full"
```

Test your Apache set up by going to `http://your_server_IP_address`. You should see the Apache2 Ubuntu Default Page!

## Install MySQL

```
$ sudo apt-get install mysql-server
```

Press `Y` to agree to all install questions.

During the installation, your server will ask you to select and confirm a password for the MySQL "root" user. This is an administrative account in MySQL that has increased privileges. Think of it as being similar to the root account for the server itself (the one you are configuring now is a MySQL-specific account, however). Make sure this is a strong, unique password, and **do not leave it blank.**

Complete the install with some security modifications:

```
$ mysql_secure_installation
```

You will be asked to enter the password you set for the MySQL root account. Next, you will be asked if you want to configure the `VALIDATE PASSWORD PLUGIN`. Enter `N` to decline.

You'll be asked to select a level of password validation. Enter `0`.

Press `Y` to agree to the rest of the install questions.

## Install PHP

```
$ sudo apt-get install php libapache2-mod-php php-mcrypt php-mysql
```

## Install ImageMagick and Unzip

```
$ sudo apt-get install unzip
$ sudo apt-get install imagemagick
```

# Step 2 - Install & Configure Omeka

Download and install Omeka's **current version** (you can check the current version by going to [Omeka.org](http://omeka.org/download/)). **Do not paste these instructions into your command line without checking that 2.5.1 is indeed the current version!!!!** If it is not the current version, find a link to the zip file of the current version. You should download this file to your home directory:

```
$ cd
$ wget http://omeka.org/files/omeka-2.5.1.zip
$ unzip omeka-2.5.1.zip
```

Move the omeka files to your web root:

```
$ sudo mv omeka-2.5.1/* /var/www/html/
$ sudo mv omeka-2.5.1/.htaccess /var/www/html/
```

Delete the old index.html file:

```
$ sudo rm /var/www/index.html
```

Make the omeka files readable by the internet:

```
$ cd /var/www/html/omeka
$ sudo chown -R www-data:www-data
```
