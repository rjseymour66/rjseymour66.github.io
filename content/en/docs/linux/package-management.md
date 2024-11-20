---
title: "Package management"
linkTitle: "Packages"
# weight: 1000
# description:
---

## dpkg

```bash
# get installed version and update status
$ dpkg -s openssh-server
Package: openssh-server
Status: install ok installed
Priority: optional
...
```

## Auto-upgrade script

Place this script in `/etc/cron.daily/`. `apt` runs as root, so use `sudo` to manually run, or use `cron`, which uses root privileges:

```bash
#!/bin/bash
# Automate regular software updates

apt update
apt upgrade -y
```

## Find an executable

```bash
# programming language
whereis go
go: /usr/local/go /usr/local/go/bin/go

# CLI
whereis aws
aws: /usr/local/bin/aws

# user binary
whereis tar
tar: /usr/bin/tar /usr/share/man/man1/tar.1.gz

```