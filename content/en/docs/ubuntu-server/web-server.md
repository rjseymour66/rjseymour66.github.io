---
title: "Web servers"
# linkTitle: ""
weight: 120
# description:
---

Web servers serve web pages to users.

## Apache

The configurations described here are unique to Ubuntu:
- Webservers display the webpage at the server IP addr by default
- config files are in `/etc/apache2`
  - main config at `/etc/apache2/apache2.conf`
- uses plugins to extend functionality
- serves content from **document root**, `/var/html/www` by default
- always reload so site doesnt go down. only restart when enabling a plugin or there are issues


You can run multiple websites on single server with _virtual hosts_
  - each virtual host has its own config file to make it unique
  - to host multiple sites, create `.conf` file in `/etc/apache2/sites-available` dir with unique `<VirtualHost>` stanza for each host
  - To run one site, you can just replace the default `index.html` from `/var/www/html`

Virtual Host setup steps:
1. Create website content
2. Upload files to server in document root?, usually `/var/www` but can be custom location
3. Make sure `www-data` owns all files in Document Root
4. Create site config file and copy into `/etc/apache2/sites-available`. For example, `mysite.com.conf`
5. Enable site with `a2ensite` (disable with `a2dissite`) - Ubuntu-specific?
   1. `a2ensite` creates a symlink to your site config file and stores in `/etc/apache2/sites-enabled`
   2. `a2dissite` deletes the symlink
6. Reload Apache to refresh config


```bash
apt install apache2             # 1. Install
systemctl status apache2        # 2. Verify its running

# ---  --- #
a2ensite mysite.com.conf        # Enable site and reload (abs path not required)
systemctl reload apache2

a2dissite mysite.com.conf       # Disable site and reload (abs path not required)
systemctl reload apache2
```

### Config files

Important config file info:
- `/etc/apache2/apache2.conf`:
  - `IncludeOptional` is where Apache looks for site `.conf` files
- VirtualHost `.conf` files are stored in `/etc/apache2/sites-available` and symlinked in `/etc/apache2/sites-enabled`
  - Each site needs its own document root
  - Default log files are fine with one site, but create custom files when serving multiple sites
  - Name-based Virtualhosts only get one IP addr, so you have to provide the `ServerName` in the config file
    - Config file example name: `000-virtual-hosts.conf`
    - Server filters incoming requests using `ServerName`, points reqs to `DocumentRoot`
    - Add as many sites as you want, just make sure your server has enough resources to handle reqs

> Research A names, domain names and how to set up Apache


```bash
# --- /etc/apache2/apache2.conf --- #
...
# Include the virtual host configurations:
IncludeOptional sites-enabled/*.conf


# --- 000-default.conf --- #
<VirtualHost *:80>                                      # listen to everything (*) on port 80
	...
	ServerAdmin webmaster@localhost                     # email addr displayed in site error messages
	DocumentRoot /var/www/html                          # serve content from this dir
	...
    # APACHE_LOG_DIR set in /etc/apache2/envvars - /var/log/ by default
	ErrorLog ${APACHE_LOG_DIR}/error.log                # Logs site errors
	CustomLog ${APACHE_LOG_DIR}/access.log combined     # Logs incoming HTTP requests
    ...
</VirtualHost>

# --- Sample VirtualHost .conf multiple interfaces --- #
<VirtualHost 10.20.30.40:80>                                        # IP addr IDs the correct network interface
    ...
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/mysitename                                # Unique DocumentRoot for each site
    ...
	ErrorLog ${APACHE_LOG_DIR}/mysitename.com-error.log             # Unique log file name    
	CustomLog ${APACHE_LOG_DIR}/mysitename.com-access.log combined  # Unique log file name
    ...
</VirtualHost>

# --- Sample name-based VirtualHost .conf single IP addr (VPS) --- #
<VirtualHost *:80>                                                  # All traffic on port 80
    ...
    ServerName firstsite.com                                        # Links incoming reqs to DocumentRoot
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/firstsite.com                             # Unique DocumentRoot for each site
    ...
	ErrorLog ${APACHE_LOG_DIR}/firstsite.com-error.log              # Unique log file name    
	CustomLog ${APACHE_LOG_DIR}/firstsite.com-access.log combined   # Unique log file name
    ...
</VirtualHost>

<VirtualHost *:80>                                                  # All traffic on port 80
    ...
    ServerName second.com                                           # Links incoming reqs to DocumentRoot
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/second.com                                # Unique DocumentRoot for each site
    ...
	ErrorLog ${APACHE_LOG_DIR}/second.com-error.log                 # Unique log file name    
	CustomLog ${APACHE_LOG_DIR}/second.com-access.log combined      # Unique log file name
    ...
</VirtualHost>
```

### Modules

Modules extend functionality. Install modules based on your needs:
- installed as modules from `apt` repo
- Debian sometimes enables modules by default
- `a2enmod`: enable a module
- `a2dismod`: disable a module

```bash
apache2 -l                                  # view built-in modules
a2enmod                                     # all modules currently installed but not enabled
apt search libapache2-mod                   # view available mods
systemctl restart apache2                   # activate a new configuration

# --- Enable module --- #
apt install libapache2-mod-php8.3           # 1. install php
a2enmod php8.3                              # 2. enable php
systemctl restart apache2                   # 3. restart service
```

### TLS

TLS encrypts your web traffic and communicates over HTTPS:
- Install signed certs into your server to protect and encrypt traffic
- SSL precedes TLS. SSL might still be in use, but use TLS for enhanced security
- apache runs on port 80, not port 443 (HTTPS)
- config file is `/etc/apache2/sites-available/default-ssl.conf`
  - `a2ensite` command enables SSL config file
- Need to generate certs:
  - self-signed: good for testing, but in prod most browsers won't trust them by default and send users to an error page first
  - certificate authority: trusted, signed by vendor. Good for prod
  - [Let's Encrypt](https://letsencrypt.org/docs/) is a good option for setting up certs

```bash
sudo ss -tulpn | grep apache                # view ports apache listens on

# --- Enable TLS --- #
a2enmod ssl                                 # 1. enable ssl mod
systemctl restart apache2                   # 2. restart service
a2ensite default-ssl.conf                   # 3. enable SSL conf file

# --- /etc/apache2/sites-available/default-ssl.conf --- #
<VirtualHost *:443>                                                     # listen on port 443
	ServerAdmin webmaster@localhost 

	DocumentRoot /var/www/html

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	SSLEngine on                                                        # enables SSL traffic

	SSLCertificateFile      /etc/ssl/certs/ssl-cert-snakeoil.pem
	SSLCertificateKeyFile   /etc/ssl/private/ssl-cert-snakeoil.key

	<FilesMatch "\.(?:cgi|shtml|phtml|php)$">                           # apply these options to files that match regex ext
		SSLOptions +StdEnvVars                                          # 
	</FilesMatch>
	<Directory /usr/lib/cgi-bin>                                        # apply options to a dir
		SSLOptions +StdEnvVars                                          # applies these options - enable default env vars
	</Directory>                                                        # for TLS
</VirtualHost>
```

#### Self-signed certs

For testing purposes:
- need directory to store certs
  - `.crt` is certificate
  - `.key` is private key

```bash
mkdir /etc/apache2/certs                                    # 1. create dir for certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \       # 2. generate certs and answer questions
-keyout /etc/apache2/certs/mysite.key \
-out /etc/apache2/certs/mysite.crt 

vim /etc/apache2/sites-avaiable/default-ssl.conf            # 3. add cert locations to conf file
SSLCertificateFile      /etc/apache2/certs/mysite.crt       #    Comment out original files, add your cert locations
SSLCertificateKeyFile   /etc/apache2/certs/mysite.key

systemctl reload apache2                                    # 4. reload service
vim /etc/apache2/sites-avaiable/default-ssl.conf            # 5. Tell apache to handle req for our site w TLS
<VirtualHost *:443>
        ServerName mydomain.com:443
        ...
```

#### CA certs

Same process as self-signed certs, but you don't generate the certs yourself, you have to request them and then copy them to your fs:
1. Create a Certificate Signing Request (CSR)
2. pay vendor's fee
3. complete necessary forms
4. prove you own the website in question
5. vendor sends you the files
6. Add them to `/ect/apache2/certs/` dir and config in `../default-ssl.conf`


#### default-ssl.conf sample

If you have multiple websites, you can copy this file as a template:
1. Copy `default-ssl.conf` and call it `/etc/apache2/sites-avaialble/mysite.conf`

```bash
<VirtualHost *:443>
	ServerName mysite.com:443
	ServerAdmin webmaster@localhost

	DocumentRoot /var/www/mysite
    ...
	ErrorLog ${APACHE_LOG_DIR}/mysite.com-error.log
	CustomLog ${APACHE_LOG_DIR}/mysite.com-access.log combined
    ...
	SSLEngine on
	...
	SSLCertificateFile      /etc/apache2/certs/mysite.crt
	SSLCertificateKeyFile   /etc/apache2/certs/mysite.key
	...
	<FilesMatch "\.(?:cgi|shtml|phtml|php)$">
		SSLOptions +StdEnvVars
	</FilesMatch>
	<Directory /usr/lib/cgi-bin>
		SSLOptions +StdEnvVars
	</Directory>
</VirtualHost>

```