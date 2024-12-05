---
title: "Web servers"
linkTitle: "Web servers"
# weight: 1000
# description:
---



## LAMP

LAMP is a common *web server* stack. A web server is software that makes local web resources available by visitors to a website, or it is the machine hosting the web service.

LAMP can stand for the following:
- Linux
- Apache web server admin software
- MySQL or MariaDB
- PHP, Perl, or Python

## Apache

Go to localhost (or the IP address for VMs) to view the default page and make sure everything is working.

- Files in `/etc/apache2`
- Manage with `systemctl`
- Logs in `/var/log/apache2`
- Opens port 80. `apt update` and `apt upgrade` make this secure enough for a test environment, but not prod

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
```

### Configuration

```bash
# apache2.conf: merge apache2.conf with .conf files in conf-enabled/
IncludeOptional conf-enabled/*.conf


# default web page location is in sites-enabled/000-default.conf
<VirtualHost *:80>
        ...
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ...
</VirtualHost>

```

### Website files

```bash
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

### Setup links

- [Canonical's How to install Apache2](https://ubuntu.com/server/docs/how-to-install-apache2)
- [Red Hat's Setting up the Apache HTTP web server](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/deploying_web_servers_and_reverse_proxies/setting-apache-http-server_deploying-web-servers-and-reverse-proxies#doc-wrapper)

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
mysql_secure_installation

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

## Security

Protect your hardware resources and data:
- Backup your environment
- Keep your packages and applications updated
- Create a staging environment for testing

### Firewalls

A firewall is a set of rules that test the contents of data packets to see if the packet should be allowed in the network:
- Firewall rules are kept in the kernel `iptables`, but each distro provides a high-level tool to manage them
  - `firewalld` on RHEL
  - `ufw` (Uncomplicated Firewall) on Debian

```bash
# config file location
/etc/default/ufw

# enable HTTP, permanent loads the rule at boot
firewall-cmd --permanent --add-port=80/tcp
success

# enable HTTPS, permanent loads the rule at boot
firewall-cmd --permanent --add-port=443/tcp
success

# apply new rules to current session
firewall-cmd --reload
success

# list all services
firewall-cmd --list-services

# shutdown a service
firewall-cmd --remove-service=<service-name>
# reload
firewall-cmd --reload


# only accept ssh from specific machine
firewall-cmd --add-rich-fule='rule family="ipv4" \
source address="10.20.30.40" port protocol="tcp" port="22" accept'

# open non-standard port
ufw allow <port-number>/tcp

# open range of non-standard ports
ufw allow <first-port>:<last-port>/tcp
```



## Encryption

You need to protect data going to and from your website:
- Puts users and your servers at risk
- Google ranks your site lower if you don't use encryption

### How CAs and encrytion work

certificate
: File that identifies the domain, owner, key, and trusted digital signature.

- A certificate lets browsers authenticate your website and verify its security. If you use HTTPS, then the browser encrypts all session data.
- All browsers have public root certificates installed--browsers can authenticate the sessions with these certs.

#### Handshake process

1. You go to a website, which means that the browser requests web files from a server. 
2. The browser requests the server's identity.
3. The server sends a copy of the certificate (that contains a public key) that it received from a trusted certificate authority (CA).
4. The browser compares the server cert against its preinstalled list of trusted public root certs
   - Cert expired?
   - Cert revoked?
5. If the server cert passes the test, then the browser encrypts a symmetric session key using the public key that the server sent and sends it back to the server
6. All data transmissions are sent using this session key

#### Historic CA process

1. Use OpenSSL to generate a key pair
2. Create a certificate signing request (CSR) that includes the public half of the key pair and website info
3. Send the CSR to a CA that reviews. If successful, they send you a certificate to install in your fs.
4. You tell the webserver where you keep the cert in the fs.

Now, you don't have to submit a CSR--you can just go to [certbot](https://certbot.eff.org/) and apply for a certificate for a registered domain.
- Certbot was created by a number of tech companies to help make the web more secure.

### Adding keys to Apache web server with certbot

1. Look at your server config file in `/etc/apache2/sites-available/000-default.conf` (Debian) or `/etc/httpd/conf/httpd.conf` (RHEL) and add your domain and port info:
   ```bash
   cat /etc/apache2/sites-available/000-default.conf 
   <VirtualHost *:80>
   	  ...
   </VirtualHost>
   
   <VirtualHost *:80>                           # listen on port 80
      ServerName bootstrap-it.com               # registered domain name
      DocumentRoot /var/www/html                
      ServerAlias www.bootstrap-it.com          # adds www prefix to domain
   </VirtualHost>
   ```
2. Go to [`certbot.eff.org`](https://certbot.eff.org/) and get instructions about how to install certbot and its dependencies.
   - certbot can read your web server config files and determine which domain to register