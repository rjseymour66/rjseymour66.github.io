+++
title = 'Disasters'
date = '2025-09-07T18:49:36-04:00'
weight = 110
draft = false
+++


You need a plan in case things break, because they will.

## General guidance

Set network shares to read-only by default.

## `git` for configuration files

Configuration files control the behavior of your apps and services. Tracking and backing them up is nearly as important as backing up your data. Git can track server configurations and uses OpenSSH by default.

### Set up the server

The server hosts the bare repository that clients push to. Run these steps on the machine that will act as your central config store:

1. Create the `/git` directory.
2. Give an admin user ownership of `/git`.
3. Initialize `/git/<repo>` as a bare repository.

The following commands walk through each step:

```bash
apt install git                         # install Git
apt install tig                         # browse commits interactively
sudo mkdir /git                         # create the git directory
chown <admin>:<admin> /git              # give admin user ownership
cd /git
git init --bare apache2                 # initialize a bare repository
```

### Set up the client

The client is the machine whose config files you want to track. For example, you might run these steps on a web server running Apache to move its config files into the repo and keep Apache pointing at the right location:

1. Create the `/git` directory.
2. Clone `/git/<repo>`.
3. Back up your existing config files.
4. Move the config files into `/git/<repo>`.
5. Delete the original config file directory.
6. Give root ownership of all files in `/git/<repo>` except `.git`.
7. Push changes to the remote as needed.

The following commands walk through each step:

```bash
sudo mkdir /git                                                         # create the git directory
git clone <ip-addr>:/git/apache2                                        # clone the repo from the server
cp -rp /etc/apache2 /etc/apache2.bak                                    # back up the config directory
mv /etc/apache2/* /git/apache2                                          # move config files into the repo
rmdir /etc/apache2                                                      # remove the original directory
find /git/apache2 -name '.?*' -prune -o -exec chown root:root {} +     # give root ownership, excluding .git
sudo ln -s /git/apache2 /etc/apache2                                    # symlink so the daemon finds the config
systemctl reload apache2                                                # reload to confirm it works
git add .
git commit -am 'message'
git push origin master

# revert a bad change
git checkout <commit-hash>              # check out the commit before the bad one
```

## Backup plan

`rsync` is the best tool for creating backups. It takes a full snapshot on the first run, then saves only the differences on subsequent runs. To get started, create an external backup drive with three directories:

- `current/` contains the current snapshot of files on the server.
- `archive/` stores files that were deleted or replaced. Pass this path to `--backup-dir`.
- `logs/` holds one log file per backup run. Redirect `rsync` output here using `$CURDATE` as the filename.

The following example backs up `/src` to the `current/` directory, moves replaced files to a dated folder in `archive/`, and logs the output:

```bash
# --backup-dir: copy files that would be deleted/replaced to a directory
#               backup-dir contains the original files, before they were replaced or deleted
CURDATE=$(date +%m-%d-%Y)
export $CURDATE
sudo rsync -avb --delete --backup-dir=/backup/archive/$CURDATE /src /backup/current > /backup/logs/$CURDATE.log
```