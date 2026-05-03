+++
title = 'Lab'
date = '2026-05-03T08:23:33-04:00'
weight = 20
draft = false
+++


This page explains how to build a Docker-based penetration testing lab from scratch.
The lab simulates a small corporate network with eight machines across two subnets: a
public-facing segment and an isolated internal segment. You use it to practice the full
offensive security lifecycle: reconnaissance, initial access, privilege escalation,
lateral movement, and exfiltration. The environment is self-contained and you can
spin up and tear down on demand.

It covers the directory structure, each machine's Dockerfile and config files, the
intentional vulnerabilities each machine introduces, and the management scripts that
deploy and control the environment.

The key files are:

```bash
lab/
├── docker-compose.yml          # network and service definitions
├── Makefile                    # make targets (deploy, teardown, rebuild, clean, status, init)
├── run.sh                      # implements make targets using docker compose
├── provision.sh                # post-deploy provisioning (iptables, WordPress install)
├── init.sh                     # full setup: Docker, containers, and third-party tools
└── machines/
    ├── Dockerfile-base         # shared base image all custom machines inherit from
    ├── p-web-01/
    │   ├── Dockerfile
    │   └── files/
    │       ├── acme-hyper-branding-5.csv
    │       └── site/
    │           ├── app.py
    │           ├── index.html
    │           ├── upload.html
    │           └── static/
    ├── p-ftp-01/
    │   ├── Dockerfile
    │   └── files/
    │       └── vsftpd.conf
    ├── p-web-02/
    │   ├── Dockerfile
    │   └── files/site/         # WordPress source tree
    ├── p-jumpbox-01/
    │   ├── Dockerfile
    │   └── files/
    │       ├── sshd_config
    │       ├── backup-user-sudo-config
    │       ├── jmartinez-user-sudo-config
    │       └── backup_data.sh
    ├── c-backup-01/
    │   ├── Dockerfile
    │   └── files/
    │       └── execute.sh
    ├── c-redis-01/
    │   ├── Dockerfile
    │   └── files/
    │       └── redis.conf
    ├── c-db-01/
    │   ├── Dockerfile
    │   └── files/
    │       ├── adminer-4.8.1.php
    │       ├── database.sql
    │       └── customers.sql
    └── c-db-02/
        └── Dockerfile
```

## Base image

All custom machines except `p-web-02`, `c-redis-01`, and `c-db-02` inherit from a shared
base image named `lab_base`. The `run.sh deploy` function builds it automatically before
building the other images. If you need to build it manually, run this command from the
`lab/` directory:

```bash
sudo docker build -f machines/Dockerfile-base -t lab_base .
```

Create `machines/Dockerfile-base`:

```bash
FROM ubuntu:24.04

# Add Default Users
RUN useradd -m -s /bin/bash jmartinez
RUN useradd -m -s /bin/bash dbrown
RUN useradd -m -s /bin/bash ogarcia
RUN useradd -m -s /bin/bash arodriguez
RUN echo "jmartinez:password123" | chpasswd
RUN echo "dbrown:wAWSD@ASw2" | chpasswd
RUN echo "ogarcia:#T9aujf8h33" | chpasswd
RUN echo "arodriguez:asucj783E@#" | chpasswd

# Add Packages
RUN apt-get update -y
RUN apt-get install -y \
                    net-tools \
                    iputils-ping \
                    iproute2 \
                    dnsutils \
                    cron \
                    vim-tiny \
                    at \
                    curl \
                    lshw \
                    sudo \
                    file \
                    ncat
```

The base image creates four users with known credentials. These accounts are targets for
credential-based attacks throughout the lab exercises.

| Username   | Password    |
| ---------- | ----------- |
| jmartinez  | password123 |
| dbrown     | wAWSD@ASw2  |
| ogarcia    | #T9aujf8h33 |
| arodriguez | asucj783E@# |

## docker-compose.yml

The compose file defines two networks with static subnets and assigns a fixed IP to every
container.

```bash
services:
  p-jumpbox-01:
    container_name: p-jumpbox-01
    hostname: p-jumpbox-01.acme-infinity-servers.com
    build:
      context: machines/p-jumpbox-01
      dockerfile: Dockerfile
    networks:
      public:
        ipv4_address: 172.16.10.13
      corporate:
        ipv4_address: 10.1.0.12

  c-backup-01:
    container_name: c-backup-01
    hostname: c-backup-01.acme-infinity-servers.com
    build:
      context: machines/c-backup-01
      dockerfile: Dockerfile
    networks:
      corporate:
        ipv4_address: 10.1.0.13
    volumes:
      - shared_vol:/mnt/scripts

  c-redis-01:
    container_name: c-redis-01
    hostname: c-redis-01.acme-infinity-servers.com
    build:
      context: machines/c-redis-01
      dockerfile: Dockerfile
    networks:
      corporate:
        ipv4_address: 10.1.0.14

  p-ftp-01:
    container_name: p-ftp-01
    hostname: p-ftp-01.acme-infinity-servers.com
    build:
      context: machines/
      dockerfile: p-ftp-01/Dockerfile
    networks:
      public:
        ipv4_address: 172.16.10.11

  p-web-01:
    container_name: p-web-01
    hostname: p-web-01.acme-infinity-servers.com
    privileged: true
    build:
      context: machines/p-web-01
      dockerfile: Dockerfile
    networks:
      public:
        ipv4_address: 172.16.10.10
    volumes:
      - shared_vol:/mnt/scripts/

  p-web-02:
    container_name: p-web-02
    privileged: true
    hostname: p-web-02.acme-infinity-servers.com
    build:
      context: machines/p-web-02
      dockerfile: Dockerfile
    volumes:
      - p_web_02_vol:/var/www/html
    networks:
      public:
        ipv4_address: 172.16.10.12
      corporate:
        ipv4_address: 10.1.0.11
    depends_on:
      - c-db-02

  c-db-02:
    container_name: c-db-02
    hostname: c-db-02.acme-infinity-servers.com
    build:
      context: machines/c-db-02
      dockerfile: Dockerfile
    volumes:
      - c_db_02_vol:/var/lib/mysql
    networks:
      corporate:
        ipv4_address: 10.1.0.16

  c-db-01:
    container_name: c-db-01
    hostname: c-db-01.acme-infinity-servers.com
    build:
      context: machines/c-db-01
      dockerfile: Dockerfile
    volumes:
      - c_db_01_vol:/var/lib/mysql
    networks:
      corporate:
        ipv4_address: 10.1.0.15

volumes:
  shared_vol:
  c_db_01_vol:
  c_db_02_vol:
  p_web_02_vol:

networks:
  public:
    name: public
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: br_public
    ipam:
      config:
        - subnet: "172.16.10.0/24"

  corporate:
    internal: true
    name: corporate
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: br_corporate
    ipam:
      config:
        - subnet: "10.1.0.0/24"
```

The compose file has three design decisions worth understanding:

- `p-ftp-01` sets `context: machines/` instead of `machines/p-ftp-01/` because its
  Dockerfile copies files from sibling machine directories (`p-web-01` and `p-web-02`).
  The build context must include those directories.
- `p-web-01` and `p-web-02` use `privileged: true` so `iptables` rules can be applied at
  runtime during provisioning.
- `p-web-01` and `c-backup-01` share `shared_vol` at `/mnt/scripts`. This lets scripts
  written by a web shell on `p-web-01` be executed by the cron job on `c-backup-01`.

## Machine Dockerfiles and config files

### p-web-01

`p-web-01` is the public-facing web server for the fictional company ACME Hyper Branding
and the initial attack target. The Flask application exposes two intentional
vulnerabilities. The upload route checks `Content-Type` but not the file extension, so
you can upload a script by spoofing the content type. The serve route renders uploaded
`.html` files as Jinja2 templates, enabling server-side template injection. The shared
volume with `c-backup-01` makes it a launchpad for lateral movement once you have a web
shell.

`machines/p-web-01/Dockerfile`:

```bash
FROM lab_base

LABEL name="p-web-01"
LABEL company="ACME Infinity Servers"

ENV FLASK_ENV=development

RUN apt-get update -y --fix-missing
RUN apt-get install software-properties-common -y
RUN add-apt-repository universe
RUN apt-get update -y
RUN apt-get install -y \
                    python3 \
                    python3-pip \
                    iptables \
                    gnupg \
                    python3-flask

WORKDIR /app

COPY files/site/static static
COPY files/site/app.py .
COPY files/site/index.html .
COPY files/site/upload.html .

RUN mkdir files
RUN mkdir uploads
COPY files/acme-hyper-branding-5.csv files/acme-hyper-branding-5.csv

RUN gpg --batch --quick-generate-key --passphrase '' arodriguez@acme-infinity-servers.com

ENTRYPOINT [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=8081", "--debug"]
```

`machines/p-web-01/files/site/app.py` is the Flask application. It exposes three routes:

- `/`: serves `index.html`
- `/upload`: accepts file uploads; checks `Content-Type` but not file extension
- `/uploads/<file_name>`: serves uploaded files; renders `.html` files as Jinja2 templates

The upload route accepts only `image/jpeg`, `image/png`, and `image/gif` content types.
The serve route renders any uploaded `.html` file as a template, which enables server-side
template injection.

```bash
import os, subprocess

from flask import (
    Flask,
    send_from_directory,
    send_file,
    render_template,
    request
)

app = Flask(__name__, template_folder=".", static_folder='static',)
app.debug = True
app.config['UPLOAD_FOLDER'] = "uploads/"

@app.route('/')
def hello_world():
    return render_template('index.html')

@app.route('/files/<path:path>')
def files(path):
    return send_from_directory('files', path)

@app.route('/upload', methods = ['GET', 'POST'])
def upload():
    msg = ''
    allowed_content_types = ['image/jpeg', 'image/png', 'image/gif']
    if request.method == 'POST':
        f = request.files['file']
        if f.content_type not in allowed_content_types:
            msg = 'File type is not allowed!'
        else:
            f.save(os.path.join(app.config['UPLOAD_FOLDER'], f.filename))
            msg = 'File upload was successful!'

    return render_template('upload.html', msg=msg)

@app.route('/uploads', methods=['GET'])
@app.route('/uploads/<path:file_name>', methods=['GET'])
def uploads(file_name):
    file_path = os.path.join('uploads', file_name)

    if not file_name:
        return '<h1>Missing file name in URL parameter.</h1>'

    if not os.path.exists(file_path):
        return '<h1>File not found.</h1>', 404

    if file_name.endswith('.html'):
        return render_template(file_name)
    elif file_name.endswith(('.jpg', '.jpeg', '.png', '.gif')):
        return send_file(file_path, mimetype='image/jpeg', as_attachment=True)
    else:
        try:
            with open(file_path, 'r') as file:
                file_content = file.read()
            return file_content
        except UnicodeDecodeError:
            return 'Cannot display binary file.'
```

---

### p-ftp-01

`p-ftp-01` simulates a company that carelessly exposed source code backups over anonymous
FTP. It is an information-gathering target used during reconnaissance. The git repositories
it hosts contain developer names, email addresses, and commit history. That data yields
credentials and internal details you can use in later attacks. The anonymous write access
also lets you upload files to the server.

`machines/p-ftp-01/Dockerfile`:

```bash
FROM lab_base

LABEL name="p-ftp-01"
LABEL company="ACME Infinity Servers"

RUN apt-get update -y --fix-missing
RUN apt-get install -y \
                    vsftpd \
                    apache2 \
                    git

RUN mkdir -p /var/www/html/backup/acme-hyper-branding
RUN mkdir -p /var/www/html/backup/acme-impact-alliance
COPY p-ftp-01/files/vsftpd.conf /etc/vsftpd.conf
COPY p-web-01/files/site/app.py /var/www/html/backup/acme-hyper-branding
COPY p-web-02/files/site/* /var/www/html/backup/acme-impact-alliance/

RUN git config --global init.defaultBranch master
RUN git init /var/www/html/backup/acme-hyper-branding \
    && cd /var/www/html/backup/acme-hyper-branding \
    && git config --global user.email "mrogers@acme-hyper-branding.com" \
    && git config --global user.name "Melissa Rogers" \
    && git add -A \
    && git commit -m 'commit code'

RUN git init /var/www/html/backup/acme-impact-alliance \
    && cd /var/www/html/backup/acme-impact-alliance \
    && git config --global user.email "kpeterson@acme-impact-alliance.com" \
    && git config --global user.name "Kevin Peterson" \
    && git add -A \
    && git commit -m 'commit code'

RUN mkdir -p /var/run/vsftpd/empty

ENTRYPOINT service cron restart \
    && \
    service apache2 restart \
    && \
    vsftpd /etc/vsftpd.conf
```

`machines/p-ftp-01/files/vsftpd.conf`:

```bash
listen=YES
anonymous_enable=YES
dirmessage_enable=YES
use_localtime=YES
connect_from_port_20=YES
write_enable=YES
xferlog_std_format=NO
log_ftp_protocol=YES
anon_root=/var/www/html
anon_upload_enable=YES
anon_mkdir_write_enable=YES
```

Anonymous FTP is enabled with write access. The `anon_root` is `/var/www/html`, so the
backup directories, including `.git` metadata, are accessible without credentials.

---

### p-web-02

`p-web-02` is the WordPress site for the fictional charity ACME Impact Alliance. It sits
on both the public and corporate subnets, making it useful as a pivot point into the
internal network once compromised. The intended attack path is to exploit WordPress after
obtaining credentials from the database on `c-db-01` or through other reconnaissance,
then use the foothold to reach corporate machines. Note that WordPress is not
pre-installed in the image. `provision.sh` completes the installation over HTTP after the
container starts.

`machines/p-web-02/Dockerfile`:

```bash
FROM wordpress:6.4

LABEL name="p-web-02"
LABEL company="ACME Infinity Servers"

ENV WORDPRESS_DB_HOST="c-db-02"
ENV WORDPRESS_DB_USER="wordpress"
ENV WORDPRESS_DB_PASSWORD="wordpress"
ENV WORDPRESS_DB_NAME="wordpress"

RUN apt-get update -y
RUN apt-get install -y \
                    net-tools \
                    socat \
                    iputils-ping \
                    iproute2 \
                    cron \
                    lshw \
                    at

ADD files/site /var/www/html

RUN chown www-data:www-data -R /var/www/html/
```

Place the WordPress source tree at `machines/p-web-02/files/site/`. Download WordPress 6.4
and extract it there, then add the custom `donate.php` file (see the book for its
contents). The `provision.sh` script completes the WordPress installation over HTTP after
the container starts.

---

### p-jumpbox-01

`p-jumpbox-01` is the bastion host that connects the public and corporate networks and
the primary pivot point for reaching internal machines. You access it by SSHing in with
credentials discovered during reconnaissance, then escalate privileges or pivot to
corporate machines using its misconfigurations. The `backup` user is the intended initial
foothold. It has a weak password, limited sudo rights that can be abused for shell
escape, and an SSH authorized-keys file you can write to via the shared volume.

`machines/p-jumpbox-01/Dockerfile`:

```bash
FROM lab_base

LABEL name="p-jumpbox-01"
LABEL company="ACME Infinity Servers"

RUN apt-get update -y --fix-missing
RUN apt-get install -y openssh-server elinks chkrootkit

RUN echo 'backup:backup' | chpasswd
RUN chsh -s /bin/bash backup

COPY files/sshd_config /etc/ssh/sshd_config
RUN chown backup:backup /var/backups

RUN chmod u+s /usr/bin/elinks

COPY files/backup-user-sudo-config /etc/sudoers.d/backup-user-sudo-config
COPY files/jmartinez-user-sudo-config /etc/sudoers.d/jmartinez-user-sudo-config

RUN mkdir scripts
RUN mkdir -p /data/backup
RUN chmod 757 /data

COPY files/backup_data.sh /scripts/backup_data.sh
RUN chmod u+x /scripts/backup_data.sh

RUN echo '*/5 * * * * root bash /scripts/backup_data.sh' >> /etc/crontab

ENTRYPOINT service ssh restart \
    && \
    service cron restart \
    && \
    tail -f /dev/null
```

`machines/p-jumpbox-01/files/sshd_config`:

```bash
Port 22
PermitRootLogin no
PasswordAuthentication yes
ChallengeResponseAuthentication no
UsePAM no

Match User backup
    AuthorizedKeysFile /var/backups/.ssh/authorized_keys
```

`machines/p-jumpbox-01/files/backup-user-sudo-config`:

```bash
backup ALL=(ALL:ALL) /usr/bin/vi
backup ALL=(ALL:ALL) /usr/bin/curl
```

`machines/p-jumpbox-01/files/jmartinez-user-sudo-config`:

```bash
jmartinez ALL=(ALL:ALL) ALL
```

`machines/p-jumpbox-01/files/backup_data.sh`:

```bash
#!/bin/bash
CURRENT_DATE=$(date +%y-%m-%d)

if [[ ! -d "/data/backup" ]]; then
    mkdir -p /data/backup
fi

for directory in "/tmp" "/data"; do
  if [[ -f "${directory}/extra_cmds.sh" ]]; then
    source "${directory}/extra_cmds.sh"
  fi
done

echo "Backing up /data/backup - ${CURRENT_DATE}"
tar czvf "/data/backup-${CURRENT_DATE}.tar.gz" /data/backup
rm -rf /data/backup/*

echo "Done."
```

Three intentional misconfigurations to note:

- `elinks` has the SUID bit set, enabling privilege escalation.
- The `backup` user can run `vi` and `curl` as root via sudo. Both can escape to a root
  shell (see GTFOBins).
- The cron job sources `extra_cmds.sh` from `/tmp` or `/data` if the file exists. Placing
  a script there achieves arbitrary code execution as root.

---

### c-backup-01

`c-backup-01` is an internal backup server reachable only from the corporate network. It
demonstrates lateral movement via the shared Docker volume. Once you have a web shell on
`p-web-01`, overwrite `execute.sh` on the shared volume. The cron job picks up the change
within a minute and runs your script, giving you remote code execution on an internal
machine without credentials.

`machines/c-backup-01/Dockerfile`:

```bash
FROM lab_base

LABEL name="c-backup-01"
LABEL company="ACME Infinity Servers"

RUN apt-get update -y --fix-missing
RUN apt-get install -y python3

COPY files/execute.sh /mnt/scripts/

RUN chmod u+x /mnt/scripts/execute.sh
RUN echo '*/1 * * * * bash /mnt/scripts/execute.sh' >> /tmp/root-crontab
RUN crontab /tmp/root-crontab && rm /tmp/root-crontab
RUN mkdir -p /var/www/site

ENTRYPOINT service cron restart && python3 -m http.server --directory /var/www/site 8080
```

`machines/c-backup-01/files/execute.sh`:

```bash
#!/bin/bash

LOG="/tmp/job.log"

echo "$(date) - Starting cleanup script..." >> "$LOG"
if find /tmp -type f ! -name 'job.log' -exec rm -rf {} +; then
    echo "cleaned up files from the /tmp folder." >> "$LOG"
fi

echo "$(date) - Cleanup script is finished." >> "$LOG"
```

The cron job runs `execute.sh` every minute as root. Because `execute.sh` lives on the
shared volume, a web shell on `p-web-01` can overwrite it to achieve remote code execution
on `c-backup-01`.

---

### c-redis-01

`c-redis-01` is an internal Redis instance that demonstrates the risk of exposing an
unauthenticated in-memory store on a network. Redis has no password and accepts writes
from any connected client. To exploit it, use Redis's `CONFIG SET` commands to change the
working directory and output filename, then write a value such as an SSH public key
directly to `/root/.ssh/authorized_keys`. That gives you root SSH access to the
container.

`machines/c-redis-01/Dockerfile`:

```bash
FROM redis:5.0.6

LABEL name="c-redis-01"
LABEL company="ACME Infinity Servers"

RUN echo "deb http://archive.debian.org/debian buster main" > /etc/apt/sources.list && \
    apt-get update -y --fix-missing
RUN apt-get install -y openssh-server

COPY --chown=root:root files/redis.conf /etc/redis/redis.conf

RUN mkdir /root/.ssh/
RUN chmod 700 /root/.ssh
RUN touch /root/.ssh/authorized_keys
RUN chmod 644 /root/.ssh/authorized_keys

ENTRYPOINT service ssh restart \
           && \
           redis-server /etc/redis/redis.conf
```

`machines/c-redis-01/files/redis.conf`:

```bash
bind 0.0.0.0
dbfilename dump.rdb
pidfile /var/run/redis_6379.pid
port 6379
protected-mode no
slave-read-only no
```

`protected-mode no` disables the default authentication requirement. `slave-read-only no`
allows writes from replicas. Together these settings let an attacker write arbitrary files
to the host. This is the classic technique for writing an SSH key to
`/root/.ssh/authorized_keys`.

---

### c-db-01

`c-db-01` is an internal database server that holds customer records for both ACME
companies, including plaintext employee passwords. Adminer runs over Apache, so anyone
who reaches the machine gets a browser-based database management interface. The intended
attack path is to reach it from the corporate network after pivoting through
`p-jumpbox-01` or `p-web-02`, then extract credentials from the `customers` database to
use against other services.

`machines/c-db-01/Dockerfile`:

```bash
FROM ubuntu:24.04

LABEL name="c-db-01"
LABEL company="ACME Infinity Servers"

ENV DEBIAN_FRONTEND=noninteractive
ARG DB_ADMINER_FILE="/var/www/html/database.sql"
ARG DB_CUSTOMERS_FILE="/var/tmp/customers.sql"

RUN apt-get update -y --fix-missing
RUN apt-get install -y \
    cron \
    mariadb-server \
    apache2 \
    php \
    php-mysql \
    lshw \
    at

COPY files/adminer-4.8.1.php /var/www/html/adminer.php
COPY files/database.sql $DB_ADMINER_FILE
COPY files/customers.sql $DB_CUSTOMERS_FILE

RUN mkdir /var/www/html/uploads
RUN chmod 777 /var/www/html/uploads

RUN service mariadb restart \
    && \
    cat "$DB_ADMINER_FILE" | mysql -u root \
    && \
    cat "$DB_CUSTOMERS_FILE" | mysql -u root && rm "$DB_CUSTOMERS_FILE"

ENTRYPOINT service mariadb restart \
    && \
    /usr/sbin/apache2ctl -D FOREGROUND
```

Download Adminer 4.8.1 from the official Adminer site and place the single PHP file at
`machines/c-db-01/files/adminer-4.8.1.php`.

`machines/c-db-01/files/database.sql` creates the Adminer database user:

```bash
CREATE DATABASE IF NOT EXISTS adminer_db;
CREATE USER IF NOT EXISTS 'adminer_user'@'localhost' IDENTIFIED BY 'P@ssword321';
GRANT ALL ON *.* TO 'adminer_user'@'localhost';
```

`machines/c-db-01/files/customers.sql` seeds two customer tables with employee records:

```bash
CREATE DATABASE IF NOT EXISTS customers;

USE customers;

CREATE TABLE acme_hyper_branding(
   id INT AUTO_INCREMENT,
   first_name VARCHAR(100),
   last_name VARCHAR(100),
   designation VARCHAR(100),
   email VARCHAR(50),
   password VARCHAR(20),
   PRIMARY KEY(id)
);

CREATE TABLE acme_impact_alliance(
   id INT AUTO_INCREMENT,
   first_name VARCHAR(100),
   last_name VARCHAR(100),
   designation VARCHAR(100),
   email VARCHAR(50),
   password VARCHAR(20),
   PRIMARY KEY(id)
);

-- acme_hyper_branding rows
INSERT INTO acme_hyper_branding VALUES (NULL,'Jacob','Taylor','Founder','jtaylor@acme-hyper-branding.com','carmen');
INSERT INTO acme_hyper_branding VALUES (NULL,'Sarah','Lewish','Executive Assistant','slewis@acme-hyper-branding.com','cachepot');
INSERT INTO acme_hyper_branding VALUES (NULL,'Nicholas','Young','Influencer','nyoung@acme-hyper-branding.com','spring2023');
INSERT INTO acme_hyper_branding VALUES (NULL,'Lauren','Scott','Influencer','lscott@acme-hyper-branding.com','gaga');
INSERT INTO acme_hyper_branding VALUES (NULL,'Aaron','Peres','Marketing Lead','aperes@acme-hyper-branding.com','aperes123');
INSERT INTO acme_hyper_branding VALUES (NULL,'Melissa','Rogers','Software Engineer','mrogers@acme-hyper-branding.com','melissa2go');

-- acme_impact_alliance rows
INSERT INTO acme_impact_alliance VALUES (NULL,'Jane','Torres','Owner','jtorres@acme-impact-alliance.com','asfim2ne7asd7');
INSERT INTO acme_impact_alliance VALUES (NULL,'Anthony','Johnson','Executive Assistant','ajohnson@acme-impact-alliance.com','3kemas8dh23');
INSERT INTO acme_impact_alliance VALUES (NULL,'David','Carter','Cat Rescuer','dcarter@acme-impact-alliance.com','asdij28ehasds');
INSERT INTO acme_impact_alliance VALUES (NULL,'Benjamin','Mitchell','Cat Rescuer','bmitchell@acme-impact-alliance.com','2rnausdiuwhd');
INSERT INTO acme_impact_alliance VALUES (NULL,'Karen','Cook','Cat Rescuer','kcook@acme-impact-alliance.com','wdnausdb723bs');
INSERT INTO acme_impact_alliance VALUES (NULL,'Kevin','Peterson','Software Engineer','kpeterson@acme-impact-alliance.com','wudhasdg72ws');
```

---

### c-db-02

`c-db-02` is the backend database for the WordPress site on `p-web-02`. It holds all
WordPress data including the `wp_users` table with hashed admin credentials. Once you
reach the corporate network, connect to it directly to read or manipulate WordPress
content, add admin users, or extract password hashes for offline cracking.

`machines/c-db-02/Dockerfile`:

```bash
FROM mariadb:10.6.4-focal

LABEL name="c-db-02"
LABEL company="ACME Infinity Servers"

ENV MYSQL_ROOT_PASSWORD="root"
ENV MYSQL_DATABASE="wordpress"
ENV MYSQL_USER="wordpress"
ENV MYSQL_PASSWORD="wordpress"

CMD ["--default-authentication-plugin=mysql_native_password"]
```

## Management scripts

### Makefile

The Makefile wraps `run.sh` for all lab operations. Create it at `lab/Makefile`:

```bash
SHELL := /bin/bash
MAKEFLAGS += --no-print-directory

.DEFAULT_GOAL := help

deploy:
	@./run.sh deploy

teardown:
	@./run.sh teardown

clean:
	@./run.sh clean

rebuild:
	@./run.sh rebuild

status:
	@./run.sh status

test:
	@./run.sh status

init:
	@./init.sh

help:
	@echo "Usage: make deploy | teardown | clean | rebuild | status | init | help"
	@echo
	@echo "deploy   | build images and start containers"
	@echo "teardown | stop containers (shutdown lab)"
	@echo "rebuild  | rebuilds the lab from scratch (clean and deploy)"
	@echo "clean    | stop and delete containers and images"
	@echo "status   | check the status of the lab"
	@echo "init     | build everything (containers and hacking tools)"
	@echo "help     | show this help message"
```

The Makefile uses a few GNU Make conventions worth knowing:

- `SHELL := /bin/bash` sets the shell for recipe commands to `bash` instead of the
  default `sh`. The `:=` operator assigns the value immediately at parse time rather
  than deferring it.
- `MAKEFLAGS += --no-print-directory` suppresses the `Entering directory` and
  `Leaving directory` messages make prints when recursing into subdirectories.
- `.DEFAULT_GOAL := help` sets `help` as the target that runs when you type `make`
  with no arguments.
- Each recipe line must be indented with a tab, not spaces. Make treats tab-indented
  lines as shell commands to execute.
- The `@` prefix on a recipe line suppresses echoing the command before running it.
  Without `@`, make prints the command and then runs it.

`make init` calls `./init.sh` directly instead of routing through `run.sh` because
`init.sh` installs Docker and third-party tools in addition to deploying containers.

### run.sh

`run.sh` implements each make target. It must run as root and logs to
`/var/log/lab-install.log`.

The `deploy` function:

1. Checks whether images are already built by counting `docker images` entries prefixed
   with `lab-` against the number of `container_name` entries in `docker-compose.yml`.
2. If images are not yet built, builds the base image, then runs
   `docker compose build --parallel`.
3. Brings containers up with `docker compose up --detach`.
4. If all containers are running, waits 25 seconds and calls `check_post_actions` from
   `provision.sh`.

The `clean` function runs `docker compose down --volumes --rmi all` followed by
`docker system prune -a --volumes -f` to remove everything.

### provision.sh

`provision.sh` runs two post-deploy steps automatically after containers start. `run.sh`
sources it. Do not run it directly.

Create `lab/provision.sh`:

```bash
#!/bin/bash

p_web_01() {
  if ! sudo docker exec -it p-web-01 bash -c \
    "iptables -I INPUT -s 10.1.0.0/24 -m comment --comment \"Block Network\" -j DROP"; then
    return 1
  fi
  return 0
}

p_web_02() {
  local result
  result=$(curl -s -X POST \
    -d 'weblog_title=ACME Impact Alliance&user_name=jtorres&first_name=Jane&last_name=Torres&admin_password=asfim2ne7asd7&admin_password2=asfim2ne7asd7&admin_email=jtorres@acme-impact-alliance.com&blog_public=0&Submit=Install WordPress&language=""' \
    "http://172.16.10.12/wp-admin/install.php?step=2")
  if ! echo "${result}" | grep -q -e "WordPress has been installed. Thank you" -e "already installed"; then
    echo "Error provisioning WordPress (p-web-02)"
    return 1
  fi
  return 0
}

check_post_actions(){
  p_web_01 || return 1
  p_web_02 || return 1
  return 0
}
```

`p_web_01` inserts an iptables rule inside the running `p-web-01` container that drops all
inbound traffic from the corporate subnet. This simulates a network boundary between the
two segments.

`p_web_02` POSTs to the WordPress installer to complete setup programmatically and checks
the response for a success string so `run.sh` can report provisioning errors.


## Tools

`init.sh` installs all tools automatically. To install them manually or verify what is
available, create a `tools` directory in your home directory first:

```bash
mkdir ~/tools
```

### Adding an alias

Some tools installed from third-party repositories such as GitHub are installed to a directory that is not on your `$PATH`, so you cannot invoke them by
name alone. An alias solves this by mapping a short name to the full command. You also use
aliases to wrap a long command (such as a `docker run` invocation) behind a simple name so
you do not have to retype it each time.

Add an alias to `~/.bashrc` so it persists across sessions:

```bash
echo "alias <name>='<command>'" >> ~/.bashrc
source ~/.bashrc
```

For example, if a binary lives at `~/tools/mytool` and is not on `$PATH`:

```bash
echo "alias mytool='~/tools/mytool'" >> ~/.bashrc
source ~/.bashrc
```

After sourcing, type `mytool` to invoke it from any directory. The sections below note when a
tool requires an alias and show the exact command to add.

### whatweb

WhatWeb fingerprints web servers and identifies the technologies a target site uses. It
sends HTTP requests and analyzes the responses, including headers, HTML content,
JavaScript includes, cookies, and meta tags, against a database of signatures called
plugins. Each plugin matches a specific technology.

WhatWeb detects CMS platforms (WordPress, Joomla, Drupal), web servers (Apache, Nginx,
IIS), JavaScript libraries, programming languages, server software, and email addresses.
Output shows each detected technology with a confidence level in brackets, for example:
`WordPress[6.4]`, `Apache[2.4.41]`, `PHP[7.4]`.

Install it with apt:

```bash
sudo apt install whatweb -y
```

Run it against a target by passing the URL or IP address:

```bash
whatweb 172.16.10.10
```

WhatWeb supports aggression levels 1 through 4. Level 1 is passive and sends a single
request. Level 4 is aggressive and follows redirects and brute-forces paths. The default
is level 1. Set the level with `--aggression`:

```bash
whatweb --aggression 3 172.16.10.12
```

Save results to a file with `--log-brief`:

```bash
whatweb 172.16.10.10 --log-brief results.txt
```

In the lab, run whatweb against `p-web-01` (`172.16.10.10:8081`) and `p-web-02`
(`172.16.10.12`) during reconnaissance to fingerprint each technology stack before
choosing an attack vector.

### rustscan

RustScan is a fast port scanner written in Rust. It acts as a front end to Nmap: it
scans all 65,535 ports in seconds, then passes the open ports it finds to Nmap for
service and version detection. This two-phase approach is much faster than running Nmap
alone across all ports.

In the lab, rustscan runs through Docker. `init.sh` pulls the image and adds an alias to
`~/.bashrc`:

```bash
alias rustscan='docker run --network=host -it --rm --name rustscan rustscan/rustscan:2.1.1'
```

After logging out and back in, type `rustscan` to invoke it.

Scan a single target across all ports:

```bash
rustscan -a 172.16.10.10
```

Pass Nmap flags for service and version detection using `--` as a separator:

```bash
rustscan -a 172.16.10.10 -- -sV -sC
```

Scan multiple targets in one command:

```bash
rustscan -a 172.16.10.10,172.16.10.11,172.16.10.12
```

Scan a specific port range and raise the file descriptor limit for speed:

```bash
rustscan -a 172.16.10.10 -r 1-10000 --ulimit 5000
```

The key flags are:

| Flag             | Description                                        |
| ---------------- | -------------------------------------------------- |
| `-a <target>`    | Target IP or hostname (required)                   |
| `-p <ports>`     | Scan specific ports, e.g. `-p 22,80,443`           |
| `-r <range>`     | Scan a port range, e.g. `-r 1-1000`                |
| `-b <size>`      | Batch size (ports per batch); default 4500         |
| `--ulimit <n>`   | Open file descriptor limit; raise for faster scans |
| `--timeout <ms>` | Per-port timeout in milliseconds                   |
| `--`             | Passes remaining arguments directly to Nmap        |

In the lab, run rustscan against the public subnet machines during reconnaissance to
discover open ports quickly before deeper enumeration with Nmap:

```bash
rustscan -a 172.16.10.10,172.16.10.11,172.16.10.12,172.16.10.13 -- -sV
```

### nuclei

Nuclei is a template-based vulnerability scanner. It runs community-maintained YAML templates
against targets to detect known vulnerabilities, misconfigurations, exposed admin panels, and
default credentials. Templates are organized by severity and category. `init.sh` installs it
with apt.

```bash
sudo apt install nuclei -y
```

Update templates before scanning:

```bash
nuclei -update-templates
```

Scan a target:

```bash
nuclei -u http://172.16.10.10:8081
```

Filter by severity:

```bash
nuclei -u http://172.16.10.12 -severity medium,high,critical
```

In the lab, run nuclei against the public web machines after initial reconnaissance to surface
known vulnerabilities and misconfigurations automatically.

### dirsearch

dirsearch brute-forces web paths to discover hidden directories and files on a target server.
It sends requests for each entry in a wordlist and reports paths that return a 2xx or 3xx
response. `init.sh` installs it with apt.

```bash
sudo apt install dirsearch -y
```

Scan a target:

```bash
dirsearch -u http://172.16.10.10:8081
```

Use a custom wordlist:

```bash
dirsearch -u http://172.16.10.10:8081 -w /usr/share/wordlists/dirb/common.txt
```

Filter by status code:

```bash
dirsearch -u http://172.16.10.12 -i 200,301,302
```

In the lab, run dirsearch against `p-web-01` (`172.16.10.10`) and `p-web-02` (`172.16.10.12`)
during reconnaissance to uncover hidden paths before exploitation.

### linux-exploit-suggester-2

Linux Exploit Suggester 2 is a Perl script that suggests local privilege escalation exploits
based on the target's kernel version. After gaining initial access to a machine, run it to
identify which known kernel exploits apply. `init.sh` clones the repository to
`~/tools/linux-exploit-suggester-2/`.

Clone it manually:

```bash
git clone https://github.com/jondonas/linux-exploit-suggester-2.git ~/tools/linux-exploit-suggester-2
```

Transfer `linux-exploit-suggester-2.pl` to a compromised machine and run it:

```bash
perl linux-exploit-suggester-2.pl
```

In the lab, use it during privilege escalation after gaining a foothold on a machine to
identify viable kernel exploit paths.

### gitjacker

Gitjacker downloads source code and commit history from exposed `.git` directories on web
servers. When a server exposes its `.git` folder, Gitjacker reconstructs the repository by
fetching individual Git objects. `init.sh` downloads the binary to `~/tools/gitjacker` and
adds an alias to `~/.bashrc`.

Install manually:

```bash
curl -s "https://raw.githubusercontent.com/liamg/gitjacker/master/scripts/install.sh" | bash
mv /usr/local/bin/gitjacker ~/tools/gitjacker
```

`init.sh` places the binary in `~/tools/` rather than a directory on `$PATH`. Add an alias so
you can invoke it by name without typing the full path:

```bash
echo "alias gitjacker='~/tools/gitjacker'" >> ~/.bashrc
source ~/.bashrc
```

Scan a target for an exposed `.git` directory:

```bash
gitjacker http://172.16.10.10:8081
```

In the lab, run gitjacker against `p-web-01` during reconnaissance to check whether the
application's `.git` directory is exposed and recoverable.

### pwncat-cs

pwncat-cs is a Python-based reverse and bind shell handler. It improves on netcat by
automatically stabilizing the remote shell into a full PTY, handling file transfers, managing
persistence, and providing a module system for enumeration and privilege escalation. `init.sh`
installs it with pip3, but that fails on modern Kali due to the externally-managed Python
environment. Use pipx instead.

```bash
sudo apt install pipx -y
pipx install pwncat-cs
pipx ensurepath
source ~/.bashrc
```

pipx creates an isolated virtual environment per tool, which is how Python CLI tools must be
installed on modern Kali.

Listen for an incoming reverse shell:

```bash
pwncat-cs -lp 4444
```

Connect to a bind shell:

```bash
pwncat-cs <target-ip> 4444
```

Press `Ctrl-D` to drop from the remote shell to the pwncat local prompt. From there you can
upload files, run built-in modules, and manage sessions.

In the lab, use pwncat-cs to catch reverse shells from compromised machines and interact with
them more effectively than a raw netcat listener.

### linenum

LinEnum is a Bash script that performs comprehensive local enumeration on a Linux system to
identify privilege escalation opportunities. It checks SUID and SGID files, sudo rights,
world-writable directories, cron jobs, running processes, network configuration, and installed
software. `init.sh` downloads it to `~/tools/LinEnum.sh` and makes it executable.

Download and make it executable manually:

```bash
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O ~/tools/LinEnum.sh
chmod u+x ~/tools/LinEnum.sh
```

Transfer `LinEnum.sh` to a compromised machine and run it:

```bash
./LinEnum.sh
```

Run a thorough scan and save output to a file:

```bash
./LinEnum.sh -t -r /tmp/linenum-report.txt
```

In the lab, transfer LinEnum.sh to a compromised machine and run it during post-exploitation
to surface privilege escalation vectors.

### unix-privesc-check

unix-privesc-check is a shell script that audits common privilege escalation vectors on Unix
systems. It checks file permissions, SUID binaries, sudo configuration, cron jobs, writable
`$PATH` entries, and service configurations. Unlike LinEnum, it produces clearly labelled
`WARN` and `OK` output for each check. It ships with Kali at `/usr/bin/unix-privesc-check`.
`init.sh` copies it to `~/tools/` for easy transfer to target machines.

Copy it to your tools directory:

```bash
cp /usr/bin/unix-privesc-check ~/tools/
```

Run a standard check on a target:

```bash
./unix-privesc-check standard
```

Run a detailed check:

```bash
./unix-privesc-check detailed
```

In the lab, transfer `unix-privesc-check` to a compromised machine alongside LinEnum and run
both to cross-reference findings during privilege escalation.