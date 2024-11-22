---
title: "Linux Apache MySQL PHP"
linkTitle: "LAMP server"
# weight: 1000
# description:
---

LAMP is a common *web server* stack. A web server is software that makes local web resources available by visitors to a website, or it is the machine hosting the web service.

LAMP can stand for the following:
- Linux
- Apache web server admin software
- MySQL or MariaDB
- PHP, Perl, or Python

## Apache

Go to localhost (or the IP address for VMs) to view the default page and make sure everything is working.
```bash
# install
apt install apache2

# view apache files
ls -logF /etc/apache2/
total 80
-rw-r--r-- 1  7178 Oct  2 12:40 apache2.conf
drwxr-xr-x 2  4096 Nov 20 01:02 conf-available/
drwxr-xr-x 2  4096 Nov 20 01:02 conf-enabled/
-rw-r--r-- 1  1782 Mar 18  2024 envvars
-rw-r--r-- 1 31063 Mar 18  2024 magic
drwxr-xr-x 2 12288 Nov 20 01:02 mods-available/
drwxr-xr-x 2  4096 Nov 20 01:02 mods-enabled/
-rw-r--r-- 1   274 Mar 18  2024 ports.conf
drwxr-xr-x 2  4096 Nov 20 01:02 sites-available/    
drwxr-xr-x 2  4096 Nov 20 01:02 sites-enabled/

# DocumentRoot controls content location - change this to
# change where Apache looks for content to serve
cat sites-enabled/000-default.conf 
<VirtualHost *:80>
	...
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html
	...
</VirtualHost>

# view default content - where Apache directs incoming browser reqs
ls /var/www/html/
index.html

# view index.html - default Apache site located at VM IP addr
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <!--
    Modified from the Debian original for Ubuntu
    Last updated: 2022-03-22
    See: https://launchpad.net/bugs/1966004
  -->
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
...
```

## MariaDB

- Data in a table is a _record_
- A _record_ is identified by a key
- A _database engine_ is a type of software that manages relational database data and lets admins and other users access it with SQL

```bash
# install
apt install mariadb-server

# check the service
systemctl status mariadb

# harden the installation
sudo mysql_secure_installation

# login as root user, prompt for passwd
mysql -u root -p 

# create db
CREATE DATABASE <db-name>;

# switch to db
use <db-name>

# create non-root user (best practice)
CREATE USER '<username>'@'localhost' IDENTIFIED BY '<password>';

# grant new user privs on table
GRANT ALL PRIVILEGES ON <db-name>.* TO '<username>'@'localhost' IDENTIFIED BY '<password>';

# refresh privs after update
FLUSH PRIVILEGES;
```

## PHP

```bash
# install
apt install php
apt install libapache2-mod-php

# restart apache2 after php install
systemctl restart apache2
```