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

This opens the global Apache configuration file in the nano text editor. Key down to the **bottom of the file** (`Ctl+V` will page down, `Ctl+Y` will page up, `Alt+/` will take you to the bottom of the file, `Alt+\` will take you to the top of the file), and add:

#### *apache2.conf*
```
ServerName server_domain_or_IP
```

(fill in the `server_domain_or_IP` with the domain name or IP of your server)

Press `Ctl+O` and `Enter` to save, then `Ctl+X` to exit.

Enable mod_rewrite globally:

```
$ sudo nano /etc/apache2/sites-enabled/000-default.conf
```

Find `DocumentRoot /var/www/html` and below this line paste in:

```
<Directory /var/www/html>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride All
    Order allow,deny
    allow from all
</Directory>

```

The edited version should look like this:

#### *000-default.conf*

```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        <Directory /var/www/html>
            Options Indexes FollowSymLinks MultiViews
            AllowOverride All
            Order allow,deny
            allow from all
        </Directory>

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

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

Delete anything that is in the `html` directory (to clear it for omeka):

```
$ sudo rm -R /var/www/html/*
```

Move the omeka files to your web root:

```
$ sudo mv omeka-2.5.1/* /var/www/html/
$ sudo mv omeka-2.5.1/.htaccess /var/www/html/
```

Make the omeka files readable by the internet:

```
$ cd /var/www/html
$ sudo chown -R www-data:www-data .
```

Enable mod_rewrite:

```
$ sudo a2enmod rewrite && service apache2 restart
```

## Set up the Database

Log in to MySQL as root:

```
$ sudo mysql -u root -p
```

(You will be prompted for your MySQL root password)

Set up the database and Omeka user:

```
mysql> CREATE DATABASE omeka CHARACTER SET utf8 COLLATE utf8_general_ci;
mysql> CREATE USER 'omeka'@'localhost' IDENTIFIED by 'PUTAPASSWORDHEREDONTJUSTCOPYTHIS';
mysql> GRANT ALL PRIVILEGES ON omeka.* TO 'omeka'@'localhost'; FLUSH PRIVILEGES;
mysql> exit;
```

Fill in your `db.ini` file:

```
$ sudo nano db.ini
```

Change this:

#### *db.ini*
```
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
; Database Configuration File ;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;
; Omeka requires MySQL 5 or newer.
;
; To configure your database, replace the X's with your specific
; settings. If you're unsure about your database information, ask
; your server administrator, or consult the documentation at
; <http://omeka.org/codex/Database_Configuration_File>.

[database]
host     = "XXXXXXX"
username = "XXXXXXX"
password = "XXXXXXX"
dbname   = "XXXXXXX"
prefix   = "omeka_"
charset  = "utf8"
;port     = ""
```

to this:

#### *db.ini*
```
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
; Database Configuration File ;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;
; Omeka requires MySQL 5 or newer.
;
; To configure your database, replace the X's with your specific
; settings. If you're unsure about your database information, ask
; your server administrator, or consult the documentation at
; <http://omeka.org/codex/Database_Configuration_File>.

[database]
host     = "localhost"
username = "omeka"
password = "password_created_during_database_setup"
dbname   = "omeka"
prefix   = "omeka_"
charset  = "utf8"
;port     = ""
```

Press `Ctl+O`, `Enter`, `Ctl+X` to save and close.

# Step 3 - Complete Installation in Browser

Go to `http://your_server_IP_address_or_domain_name/install`

If everything worked, you should see an installation set-up form!! Complete the form and install your new site.

## Test your ImageMagick path

After your site has installed, go to "Settings" in the navigation bar at the top of the admin Dashboard (http://your_server_IP_address_or_domain_name/admin/).

Scroll down to "ImageMagick Directory Path" and click on "Test". If this does not work, you will need to find your path. It is usually `/usr/bin`
