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

### ldd

Check which libraries a program uses:

```bash
# find the executable
which ssh
/usr/bin/ssh

# check the libraries it uses
ldd /usr/bin/ssh
	linux-vdso.so.1 (0x00007ffc92ab8000)
	libcrypto.so.3 => /lib/x86_64-linux-gnu/libcrypto.so.3 (0x0000751246200000)
	libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007512461e4000)
	...
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

Core tool is `apt`:
- `apt-cache`: 
- `apt-get`: installs, updates, and removes packages
- `apt`: front end script that can call either `apt-cache` or `apt-get`

The `/etc/apt/sources.list` file contains address of other repos that the `apt` tool is configured to use.

```bash
# view nonstandard repositories
grep -v "#" /etc/apt/sources.list

deb http://gb.archive.ubuntu.com/ubuntu/ jammy main restricted

deb http://gb.archive.ubuntu.com/ubuntu/ jammy-updates main restricted

deb http://gb.archive.ubuntu.com/ubuntu/ jammy universe
deb http://gb.archive.ubuntu.com/ubuntu/ jammy-updates universe

deb http://gb.archive.ubuntu.com/ubuntu/ jammy multiverse
deb http://gb.archive.ubuntu.com/ubuntu/ jammy-updates multiverse

deb http://gb.archive.ubuntu.com/ubuntu/ jammy-backports main restricted universe multiverse

deb http://security.ubuntu.com/ubuntu jammy-security main restricted
deb http://security.ubuntu.com/ubuntu jammy-security universe
deb http://security.ubuntu.com/ubuntu jammy-security multiverse

```

### apt-cache

Provides info about the package database. Useful command options:

```bash
depends # displays dependencies required for the package
pkgnames # displays all packages installed on system
showpkg # displays info about specified package
stats # displays package stats for the system
unmet # displays any unmet dependencies for the installed packages
```

### apt-get

```bash
# check for broken dependencies
apt-get check
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done

# clean up database and any tempory download files
apt-get clean

# retrieve updated information about pacakges in the repo
apt-get update
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

### Red Hat repository tools

- Previous tool was `yum` (YellowDog Update Manager), developed for YellowDog Linux distro
- Replaced by `dnf`, which is updated version of `yum`
  - Query, install, and remove software packages on your system
- `dnf` and `yum` use `/etc/yum.repos.d/` directory to hold fils that list the different repos to check for packages
  - files in this folder contain URL of repo and location of additional package files within the repo

### dnf

```bash
dnf <action> <program>
alias # define alias that points to list of other dnf commands
autoremove # rm any unneeded packages installed as dependeney
check # examine local package db and report problems
check-update # check repo for update to specified package
clean # perform cleanup of temporary files kept for repos
deplist # deprecated alisas for the repoquery command
distro-sync # downgrade or install packages to place the system in sync w current repos
downgrade # downgrade package to version in the repo
group # manage a set of packages as a single entity
help # displays help
history # display previous dnf commands
info # info about installed and available packages
install # install current version of package
list # displays installed and available packages
makecache # download metadata for the repos
mark # marks specified package as installed
module # manages module packages
provides # displays package that installed specified file
reinstall # attempts to reinstall the specified package
remove # rms the specified package, including packages that depend on it
repoinfo # displays info about the configured repo
repolist # displays a list of currently configured repos
repoquery # searches the configured repos for the specified package
repository-packages # runs commands on all packages in the repo
search # search pacakge metadata for specified keywords
shell # display interactive shell for entering multiple dnf commands
swap # rm and install the specified package
updateinfo # display update advisory msgs
upgrade # install latest version of specified packages, or all pkgs if none are specified
upgrade-minimal # install only latest package versions that provide bugfix or security fix
```

## Application containers

- Containers bundle all files required for an application, including dependencies, into one distro package (the container)
- Each app has exactly the correct dependencies and versions
  - Caveat: dependencies shared among multiple applications are duplicated for each application

### snap containers

- Created and maintained by Canonical
- `snap` is an application container format 
- `snapd` app manages the snap packages
- has CLI tool:

```bash
# get snap version
snap version
snap    2.61.3+22.04
snapd   2.61.3+22.04
series  16
ubuntu  22.04
kernel  5.15.0-91-generic

# list currently installed snap packages
snap list
Name                       Version           Rev    Tracking         Publisher      Notes
bare                       1.0               5      latest/stable    canonical✓     base
chromium                   123.0.6312.86     2805   latest/stable    canonical✓     -
core                       16-2.61.2         16928  latest/stable    canonical✓     core
...

# find specific package
snap find cups
Name                              Version                Publisher             Notes  Summary
cups                              2.4.7-8                openprinting✓         -      The CUPS Snap - The Printing Stack for Linux
musescore                         3.6.2                  musescore✓            -      Create, play and print beautiful sheet music.
...
# install snap package
sudo snap install stress-ng

# remove snap pacakge
sudo snap remove stress-ng
snap "stress-ng" is not installed

# disable
sudo snap disable <package>

# enable
sudo snap enable <package>
```

### flatpak containers

- Opensource, not tied to a specific distro
- RHEL uses often uses this

```bash
# list packages
flatpack list

# add remote (repository)
sudo flatpak remote-add --if-not-exists flathub

# search for application
sudo flatpack search mosh

# install package with application ID 
sudo flatpak install org.mosh.mosh

# unistall package
sudo flatpak uninstall org.mosh.mosh
```

## Analyzing application dependencies

### Versioning

Versioning is the management of multiple application software updates through a numbering process.