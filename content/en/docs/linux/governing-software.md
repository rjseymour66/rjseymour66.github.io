---
title: "Governing software"
weight: 120
description: >
  "Installing and managing software"
---

## Working with source code

Source code projects can consist of many types of files:
- Source code files
- Header files
- Library files
- Documentation files

### wget

CLI toll that retrieves files from remote servers using FTP, FTPS, HTTP, or HTTPS:

```bash
wget http://remotehost/filename
```

- [Phoenixnap](https://phoenixnap.com/kb/wget-command-with-examples)
- [Linuxize](https://linuxize.com/post/wget-command-examples/)

### cURL

Same as `wget` but supports more protocols. Also warns if the server is using a self-signed cert or if the cert is signed by an untrusted authority.

### tar

Bundles project files into a single output file for easy transfer across the network. Also preserves folder structure and ownership. Some common option groups:

```bash
# Create a new tar file
tar -cvf test.tar test1.txt test2.txt test3.txt 
test1.txt
test2.txt
test3.txt
# Display contents of tar file
tar -tvf test.tar 
-rw-rw-r-- ryanseymour/ryanseymour 14 2024-04-07 08:48 test1.txt
-rw-rw-r-- ryanseymour/ryanseymour 14 2024-04-07 08:48 test2.txt
-rw-rw-r-- ryanseymour/ryanseymour 16 2024-04-07 08:48 test3.txt
# Extract contents of tar file
tar -xvf test.tar 
test1.txt
test2.txt
test3.txt
ll -a
total 32
drwxrwxr-x 2 ryanseymour ryanseymour  4096 Apr  7 08:52 ./
drwxrwxr-x 8 ryanseymour ryanseymour  4096 Apr  7 08:46 ../
-rw-rw-r-- 1 ryanseymour ryanseymour    14 Apr  7 08:48 test1.txt
-rw-rw-r-- 1 ryanseymour ryanseymour    14 Apr  7 08:48 test2.txt
-rw-rw-r-- 1 ryanseymour ryanseymour    16 Apr  7 08:48 test3.txt
-rw-rw-r-- 1 ryanseymour ryanseymour 10240 Apr  7 08:49 test.tar

# decompress tarball (-z option preserves the tar.gz file)
tar -zxvf test.tar.gz 
test1.txt
test2.txt
test3.txt
ll -a
total 24
drwxrwxr-x 2 ryanseymour ryanseymour 4096 Apr  7 09:10 ./
drwxrwxr-x 8 ryanseymour ryanseymour 4096 Apr  7 08:46 ../
-rw-rw-r-- 1 ryanseymour ryanseymour   14 Apr  7 08:48 test1.txt
-rw-rw-r-- 1 ryanseymour ryanseymour   14 Apr  7 08:48 test2.txt
-rw-rw-r-- 1 ryanseymour ryanseymour   16 Apr  7 08:48 test3.txt
-rw-rw-r-- 1 ryanseymour ryanseymour  193 Apr  7 08:49 test.tar.gz
```

## Compiling source code

You need to install the GNU C compiler (`gcc`) on fresh installs:
- Debian: install the `build-essential` program
- Rocky: install `Development Tools` program

```bash
# Debian
sudo apt update && sudo apt install build-essential
```

### make

Allows developers to create scripts that guide compiling and installation process of application source code:

1. Run `configure`, which analyzes the Linux system and customizes the `make` script.
2. Run `make` to build library files and executable.
3. Run `make install` as root to install the application files in the correct directories on your system.

## Packaging applications

A package bundles already compiled applications for distribution. It consists of all files require to run a single application:
- Tracking these packages is called _package management_
- Linux tracks installed packages, and their files and file locations with a database

Popular package management tools:
- Debian package management
- Red Hat package management

These tools track the following:
- Application files: the files and file locations
- Library dependencies: What library files are required for each app, and can warn you if a dependent library file is not present
- Application version: So you know when an updated version of the app is available

## Debian package tools

- Debian systems bundle application files into a single DEB package
- Core tool is `dpkg` program
- Can find DEB versions of application packages on the app website or central location for the distro

### dpkg

Install, update, and remove DEB packages:
- action is action to be taken on the file
- each action has a set of options that modify its behavior, such as force overwrting

```bash 
dpkg [options] action package-file
-C # searches for broken installed pacakges and suggests fixes
--configure # reconfigures broken package
--get-selections # displays currently installed packages
-i # installs the package
-I # displays info about uninstalled package file
-l # lists all installed packages matching a specified pattern
-L # lists installed files associated with a package
-p # displays information about an installed package
-P # rms installed package and config files
-r # rm installed package but leaves config files (good for reinstalls)
-S # locates package that owns the specified files

# list all installed packages
dpkg -l
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                                  Version                                  >
+++-=====================================================-=========================================>
ii  accountsservice                                       22.07.5-2ubuntu1.5                       >
ii  acl                                                   2.3.1-1                                  >
ii  acpi-support                                          0.144                                    >
ii  acpid                                                 1:2.0.33-1ubuntu1   
...

# list packages using a search term
dpkg -l openssh*
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                Version            Architecture Description
+++-===================-==================-============-===========================================>
ii  openssh-client      1:8.9p1-3ubuntu0.6 amd64        secure shell (SSH) client, for secure acces>
ii  openssh-server      1:8.9p1-3ubuntu0.6 amd64        secure shell (SSH) server, for secure acces>
ii  openssh-sftp-server 1:8.9p1-3ubuntu0.6 amd64        secure shell (SSH) sftp server module, for >
un  openssh-sk-helper   <none>             <none>       (no description available)
```

## Red Hat package tools

- Red Hat-based systems bundle application files into a single RPM package
- Core tool is `rpm` program

### rpm

```bash
rpm action [options] package-file
-b # builds binary from source files
-e # uninstalls specified package
-F # upgrades a package if an earlier version exists
-i # installs package
-q # queries if the specified package is installed
-U # installs or upgrades the package
-V # verifies if the package files are present
```

## Repositories

If you use `dpkg` or `rpm`, you have to find the packages to install them and any dependencies. Distros solve this problem with _repositories_, which contain software apcakges that have been tested and known to install and work correctly in the distro environment.

### Debian repository tools

Core tool is `apt`. The `/etc/apt/sources.list` file contains address of other repos that the `apt` tool is configured to use.
- `apt-cache`: 
- `apt-get`: installs, updates, and removes packages
- `apt`: front end script that can call either `apt-cache` or `apt-get`

### apt-cache

Provides info about the package database. Useful command options:

```bash
depends # displays dependencies required for the package
pkgnames # displays all packages installed on system
showpkg # displays info about specified package
stats # displays package stats for the system
unmet # displays any unmet dependencies for the installed packages
```

### apt

Installs and removes pacakges. Has the following options:

```bash
apt action program
autoremove # rm any unneeded packages automatically installed as a dependency of another installed package
full-upgrade # same as upgrade but removes any installed packages req'd to upgrade entire system
install # installs new package from repository
list # displays currently installed packages
purge # rms specified applciation and any config or data files
reinstall # reinstalls existing package from the repo
remove # rm application but keep config and data files
satisfy # resolve software dependencies in the installed packages
search # search for a specific package in the repo
show # displays info about the package
update # downloads package info from all configured repositories
upgrade # installs available upgrades from all installed packages
```