---
title: "Preventing disasters"
linkTitle: "Disasters"
weight: 210
# description:
---

You need a plan in case things break, because they will.

## General guidance

- Network shares: Set these to read-only, by default.

## Configuration file management with git

The config files control the behavior of your apps and services, so tracking changes and backing them up is important:
- Configuration is almost as important as the data you store on your server
- Git can keep track of server configurations
- Git uses OpenSSH by default

Overview steps:
Server:
1. Create /git dir
2. Give git admin privs to /git
3. Init /git/<repo> as a bare repo

Client:
1. Create /git dir
2. Clone /git/<repo>
3. Make a backup of config files
4. Move files to /git/<repo>
5. Delete original config file dir
6. Give root ownership of all files in /git/<repo> except for `.git`
7. Make changes a push to remote as needed


```bash
apt install git                 # install package
apt install tig                 # helps browse through commits

# git server
sudo mkdir /git                         # 1. Create git dir in root
chown <admin>:<admin> /git              # 2. Change owner to admin w/SSH privs
cd /git                                 # 3. Change to git dir
git init --bare apache2                 # 4. Create bare git repo

# go to webserver (client)
sudo mkdir /git                         # 1. Create git dir in root
git clone <ip-addr>:/git/apache2        # 2. Clone repo
cp -rp /etc/apache2 /etc/apache2.bak    # 3. Back up apache config dir
mv /etc/apache2/* /git/apache2          # 4. Move config files to git repo
rmdir /etc/apache2                      # 5. Remove original apache dir
find /git/apache2 -name '.?*' -prune -o -exec chown root:root {} +      # 7. root owns apache2/ dir, except .git/
sudo ln -s /git/apache2 /etc/apache2    # 8. Create symlink in /etc so apache2 daemon can find config files
systemctl reload apache2                # 9. Reload apache and confirm it works
git add .                               # 10. Add all files to staging area
git commit -am 'message'                # 11. Commit the changes
git push origin master                  # 12. Push to remote

# --- Revert changes --- #
git checkout <commit-hash>              # 1. Checkout the commit BEFORE the bad commit
```

## Backup plan

`rsync` is the best tool - lets you create backup files and then save the differentials for incremental backups:
- Create an external backup drive with these three folders:
  - `current/`: current snapshot of files on server
  - `archive/`: set this as the `--backup-dir=/path/to/archive` to store deleted/replaced files
  - `logs/`: logs from each backup. Redirect `rsync` output to a file here. Ex: `rsync -avb ... > /logs/$CURDATE.log`
- 

```bash
# --backup-dir: copy files that would be deleted/replaced to a directory
#               backup-dir contains the original files, before they were replaced or deleted
CURDATE=$(date +%m-%d-%Y)
export $CURDATE
sudo rsync -avb --delete --backup-dir=/backup/archive/$CURDATE /src /backup/current > /backup/logs/$CURDATE.log
```