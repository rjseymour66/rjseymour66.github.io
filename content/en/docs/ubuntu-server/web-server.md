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
- uses plugins for ...
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