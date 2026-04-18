+++
title = 'Web Servers'
date = '2025-09-07T19:01:06-04:00'
weight = 10
draft = false
+++


## Apache

The configurations described here are specific to Ubuntu. Apache serves content from
the **document root** (`/var/www/html` by default) and uses modules to extend functionality.

Follow these guidelines when managing Apache:

- Reload the service after configuration changes to avoid downtime. Restart only when enabling a new module or resolving an issue.
- Configuration files are in `/etc/apache2`. The main configuration file is `/etc/apache2/apache2.conf`.

You can run multiple websites on a single server using virtual hosts. Each virtual host
has its own configuration file in `/etc/apache2/sites-available`. To run a single site,
replace the default `index.html` in `/var/www/html`.

### Set up a virtual host

Use these steps to configure Apache to serve a new site:

1. Create your website content.
2. Upload the files to the document root, usually `/var/www`, or a custom location.
3. Make sure `www-data` owns all files in the document root.
4. Create a site configuration file in `/etc/apache2/sites-available` with a unique `<VirtualHost>` stanza for each host. For example, `mysite.com.conf`.
5. Enable the site with `a2ensite` and disable it with `a2dissite`. These commands create and delete a symlink in `/etc/apache2/sites-enabled`.
6. Reload Apache to apply the configuration.


### Install Apache

Install Apache and verify it is running:

1. Install the package.
   ```bash
   apt install apache2
   ```
2. Verify the service started.
   ```bash
   systemctl status apache2
   ```

Enable or disable a site by name. An absolute path is not required:

```bash
a2ensite mysite.com.conf
systemctl reload apache2

a2dissite mysite.com.conf
systemctl reload apache2
```

### Configuration files

Apache configuration files are in `/etc/apache2`:

| File | Description |
|:---|:---|
| `apache2.conf` | Main configuration file. The `IncludeOptional` directive tells Apache where to look for site `.conf` files. |
| `sites-available/` | Stores virtual host configuration files. Each site needs its own document root. |
| `sites-enabled/` | Contains symlinks to active site configuration files. |

When serving multiple sites, create a unique log file for each. Name-based virtual hosts share a single IP address, so each configuration file must include a `ServerName` directive. Apache uses `ServerName` to route incoming requests to the correct `DocumentRoot`. A common convention for the configuration file name is `000-virtual-hosts.conf`.


#### apache2.conf

The `IncludeOptional` directive tells Apache where to load site configuration files:

```apacheconf
# /etc/apache2/apache2.conf
IncludeOptional sites-enabled/*.conf
```

#### 000-default.conf

The default virtual host listens on all interfaces on port 80 and serves content from `/var/www/html`:

```apacheconf
<VirtualHost *:80>
    ServerAdmin webmaster@localhost                     # email address displayed in error messages
    DocumentRoot /var/www/html                          # serve content from this directory
    # APACHE_LOG_DIR is set in /etc/apache2/envvars — /var/log/ by default
    ErrorLog ${APACHE_LOG_DIR}/error.log                # logs site errors
    CustomLog ${APACHE_LOG_DIR}/access.log combined     # logs incoming HTTP requests
</VirtualHost>
```

#### Multiple interfaces

Bind each virtual host to a specific IP address to identify the correct network interface:

```apacheconf
<VirtualHost 10.20.30.40:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/mysitename
    ErrorLog ${APACHE_LOG_DIR}/mysitename.com-error.log
    CustomLog ${APACHE_LOG_DIR}/mysitename.com-access.log combined
</VirtualHost>
```

#### Name-based virtual hosts

Use a single IP address and route requests by `ServerName`. Each site requires a unique `DocumentRoot` and log files:

```apacheconf
<VirtualHost *:80>
    ServerName firstsite.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/firstsite.com
    ErrorLog ${APACHE_LOG_DIR}/firstsite.com-error.log
    CustomLog ${APACHE_LOG_DIR}/firstsite.com-access.log combined
</VirtualHost>

<VirtualHost *:80>
    ServerName second.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/second.com
    ErrorLog ${APACHE_LOG_DIR}/second.com-error.log
    CustomLog ${APACHE_LOG_DIR}/second.com-access.log combined
</VirtualHost>
```

### Modules

Apache modules extend functionality. Install them from the `apt` repository. Debian
sometimes enables modules by default. Use `a2enmod` to enable a module and `a2dismod`
to disable one.

Use these commands to explore available modules:

```bash
apache2 -l                      # list built-in modules
a2enmod                         # list modules available to enable
apt search libapache2-mod       # search available modules in apt
```

To enable a module:

1. Install the package.
   ```bash
   apt install libapache2-mod-php8.3
   ```
2. Enable the module.
   ```bash
   a2enmod php8.3
   ```
3. Restart the service to activate the change.
   ```bash
   systemctl restart apache2
   ```

### TLS

TLS encrypts web traffic over HTTPS. Apache listens on port 80 by default. Enabling TLS
adds a listener on port 443. The TLS configuration file is
`/etc/apache2/sites-available/default-ssl.conf`.

SSL preceded TLS. You may still see SSL references in configuration files, but use TLS
for all new deployments.

You must provide a certificate to enable TLS. Choose the type based on your environment:

| Type | Use case |
|:---|:---|
| Self-signed | Testing only. Most browsers will display a warning before allowing access. |
| Certificate authority | Production. Trusted and signed by a vendor. |
| [Let's Encrypt](https://letsencrypt.org/docs/) | Production. Free, automated, and widely trusted. |

To verify which ports Apache is listening on:

```bash
sudo ss -tulpn | grep apache
```

#### Enable TLS

Enable the SSL module, restart Apache, and activate the SSL configuration file:

1. Enable the SSL module.
   ```bash
   a2enmod ssl
   ```
2. Restart the service.
   ```bash
   systemctl restart apache2
   ```
3. Enable the SSL configuration file.
   ```bash
   a2ensite default-ssl.conf
   ```

#### default-ssl.conf

The default SSL configuration file listens on port 443. The `snakeoil` certificate is a
self-signed placeholder installed by default on Ubuntu. Replace it with your own certificate
before deploying to production:

```apacheconf
<VirtualHost *:443>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    SSLEngine on

    SSLCertificateFile      /etc/ssl/certs/ssl-cert-snakeoil.pem
    SSLCertificateKeyFile   /etc/ssl/private/ssl-cert-snakeoil.key

    <FilesMatch "\.(?:cgi|shtml|phtml|php)$">
        SSLOptions +StdEnvVars      # apply SSL environment variables to dynamic files
    </FilesMatch>
    <Directory /usr/lib/cgi-bin>
        SSLOptions +StdEnvVars      # apply SSL environment variables to CGI scripts
    </Directory>
</VirtualHost>
```

#### Self-signed certificates

Use self-signed certificates for testing. A self-signed certificate uses a `.crt` file
for the certificate and a `.key` file for the private key.

1. Create a directory for the certificates.
   ```bash
   mkdir /etc/apache2/certs
   ```
2. Generate the certificate and key. Answer the prompts when asked.
   ```bash
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
   -keyout /etc/apache2/certs/mysite.key \
   -out /etc/apache2/certs/mysite.crt
   ```
3. Update `default-ssl.conf` to point to your certificate files. Replace the existing `SSLCertificateFile` and `SSLCertificateKeyFile` lines:
   ```apacheconf
   SSLCertificateFile      /etc/apache2/certs/mysite.crt
   SSLCertificateKeyFile   /etc/apache2/certs/mysite.key
   ```
4. Add a `ServerName` directive to the `<VirtualHost>` block:
   ```apacheconf
   <VirtualHost *:443>
       ServerName mydomain.com:443
       ...
   ```
5. Reload Apache to apply the changes.
   ```bash
   systemctl reload apache2
   ```

#### CA certificates

CA certificates follow the same process as self-signed certificates, but the vendor
generates the certificate files instead of you. Once you receive the files, add them
to `/etc/apache2/certs/` and update `default-ssl.conf` to reference them.

To obtain a CA certificate:

1. Create a Certificate Signing Request (CSR).
2. Pay the vendor's fee.
3. Complete the required forms.
4. Prove you own the domain.
5. The vendor sends you the certificate files.
6. Copy the files to `/etc/apache2/certs/` and update `/etc/apache2/sites-available/default-ssl.conf`.


#### default-ssl.conf sample

Use `default-ssl.conf` as a template when hosting multiple sites over TLS. Copy it and
give it a site-specific name:

```bash
cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/mysite.conf
```

Then update the new file with your site's values:

```apacheconf
<VirtualHost *:443>
    ServerName mysite.com:443
    ServerAdmin webmaster@localhost

    DocumentRoot /var/www/mysite

    ErrorLog ${APACHE_LOG_DIR}/mysite.com-error.log
    CustomLog ${APACHE_LOG_DIR}/mysite.com-access.log combined

    SSLEngine on

    SSLCertificateFile      /etc/apache2/certs/mysite.crt
    SSLCertificateKeyFile   /etc/apache2/certs/mysite.key

    <FilesMatch "\.(?:cgi|shtml|phtml|php)$">
        SSLOptions +StdEnvVars
    </FilesMatch>
    <Directory /usr/lib/cgi-bin>
        SSLOptions +StdEnvVars
    </Directory>
</VirtualHost>
```

## nginx

nginx functions as both a web server and a proxy server. A proxy server sits between
clients and backend servers, forwarding requests and returning responses on their behalf.
Do not run nginx and Apache on the same machine — port conflicts will prevent both
services from starting correctly.

nginx uses the same `sites-available` and `sites-enabled` directory structure as Apache,
but has no `a2ensite`-style commands. You must create symlinks manually to enable sites.
Configuration files are in `/etc/nginx` and content is served from `/var/www/html`.

For a single site, replace the content of `/etc/nginx/sites-available/default`. For
multiple sites, copy the default configuration file, make your changes, and create a
new directory in `/var/www/`.

### Install nginx

Install nginx and verify it is running:

1. Install the package.
   ```bash
   apt install nginx
   ```
2. Verify the service started.
   ```bash
   systemctl status nginx
   ```

To enable a site, create the symlink manually and reload the service:

```bash
sudo ln -s /etc/nginx/sites-available/site.com /etc/nginx/sites-enabled/site.com
systemctl reload nginx
```

### Multiple site configuration

Each additional site requires its own configuration file. The `default` configuration
file in `sites-available` includes a template at the end you can use as a starting point.

1. Copy the default configuration file.
   ```bash
   cp /etc/nginx/sites-available/default /etc/nginx/sites-available/mysite.com
   ```
2. Edit the new configuration file. Remove the `default_server` flag from the `listen`
   directives, since only one site can be the default. Update `root` and `server_name`
   for your site:
   ```nginx
   server {
       listen 80;
       listen [::]:80;

       root /var/www/mysite.com;

       index index.html index.htm index.nginx-debian.html;

       server_name mysite.com www.mysite.com;

       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```
### TLS

nginx TLS requires a PEM-format certificate file and a separate private key file. PEM
(Privacy Enhanced Mail) is a Base64-encoded format for storing cryptographic keys and
certificates. It is the most common certificate format used by web servers. Store the
files in a dedicated directory and update your site configuration to listen on port 443.

1. Create a directory for the certificates.
   ```bash
   mkdir /etc/nginx/certs
   ```
2. Generate a self-signed certificate and key. Answer the prompts when asked.
   ```bash
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
   -keyout /etc/nginx/certs/mysite.key \
   -out /etc/nginx/certs/mysite.crt
   ```
3. Update your site configuration to listen on port 443 and reference the certificate files:
   ```nginx
   server {
       listen 443 ssl;
       listen [::]:443 ssl;

       root /var/www/tlssite.com;

       index index.html index.htm index.nginx-debian.html;

       server_name tlssite.com www.tlssite.com;

       ssl_certificate     /etc/nginx/certs/mysite.crt;
       ssl_certificate_key /etc/nginx/certs/mysite.key;
       ssl_session_timeout 5m;

       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```
4. Update the default site configuration to redirect HTTP traffic to HTTPS:
   ```nginx
   server {
       listen 80 default_server;
       listen [::]:80 default_server;

       return 301 https://$host$request_uri;
   }
   ```

## Nextcloud

Nextcloud is a self-hosted file sync and collaboration platform. It requires a web server
to run. Use it to sync files between machines, manage contacts and tasks, and retrieve
email from a server.

### Install Nextcloud

Download, extract, and configure Nextcloud to run under Apache:

1. Download and extract the Nextcloud archive.
   ```bash
   wget https://download.nextcloud.com/server/releases/latest.zip
   unzip latest.zip
   ```
2. Move the extracted directory to the Apache document root.
   ```bash
   mv nextcloud /var/www/html/nextcloud
   ```
3. Set ownership so Apache can read and write the files.
   ```bash
   chown www-data:www-data -R /var/www/html/nextcloud
   ```
4. Create and enable the Apache site configuration.
   ```bash
   vim /etc/apache2/sites-available/nextcloud.conf
   a2ensite nextcloud.conf
   ```
5. Install the required PHP packages and restart Apache.
   ```bash
   apt install libapache2-mod-php8.3 php8.3-curl php8.3-gd php8.3-intl \
               php8.3-mbstring php8.3-mysql php8.3-xml php8.3-zip
   systemctl restart apache2
   ```

### Set up the database

1. Log in to MariaDB as root and create the Nextcloud database and user.
   ```bash
   mariadb -u root -p
   CREATE DATABASE nextcloud;
   GRANT ALL ON nextcloud.* TO 'nextcloud'@'localhost' IDENTIFIED BY 'super_secret_password';
   ```

### Complete web setup

Navigate to `http://<server-ip>/nextcloud` and complete the setup form:

| Field | Value |
|:---|:---|
| New admin account | admin |
| New admin password | Your chosen password |
| Database account | nextcloud |
| Database password | Your chosen password |
| Database name | nextcloud |
| Database host | localhost (or the database server IP if hosted separately) |

Select **Install**, then create a regular user account. Use the admin account to enable or disable apps.

### Configuration file

The Nextcloud Apache configuration block is accessible at `www.<domain-name>.com/nextcloud`. If `ServerName` is not set, it falls back to `<server-ip>/nextcloud`:

```apacheconf
<Directory /var/www/html/nextcloud/>
    Options +FollowSymlinks
    AllowOverride All

    <IfModule mod_dav.c>
        Dav off
    </IfModule>

    SetEnv HOME /var/www/html/nextcloud
    SetEnv HTTP_HOME /var/www/html/nextcloud
</Directory>
```