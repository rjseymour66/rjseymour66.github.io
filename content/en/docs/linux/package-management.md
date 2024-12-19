---
title: "Package management"
linkTitle: "Packages"
# weight: 1000
# description:
---

## dpkg

Install, remove, and build packages, but cannot download and install packages or their dependencies.
- Older version of apt

```bash
dpkg -s openssh-server        # get installed version and update status
Package: openssh-server
Status: install ok installed
Priority: optional


dpkg -l                 # list packages
dpkg -L <package>       # list all file locations for package
dpkg -i <file.deb>      # install a deb file
dpkg -r <package-name>  # uninstall a package

# complete the configuration process for all packages that are 
# unpacked but not fully configured
dpkg --configure -a
```

neofetch --version
Neofetch 7.1.0


## apt tools

Use `apt`.

Core tool is the Advanced Packaging Tool (`apt`):
- `apt-cache`: 
- `apt-get`: installs, updates, and removes packages
- `apt`: front end script that can call either `apt-cache` or `apt-get`
- `sudo aptitude`: opens an `apt` GUI in the terminal
- actions logged in `/var/log/dpkg.log`

`apt-get` and `apt-cache` have a lot of low-level commands that were not commonly used. `apt` consists of the most widely used features of `apt-get` and `apt-cache`:
- `apt` can install new packages or the kernel, but `apt-get` cannot

The `/etc/apt/sources.list` file contains address of other repos that the `apt` tool is configured to use:

```bash
cat /etc/apt/sources.list.d/ubuntu.sources
Types: deb
URIs: http://us.archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

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
depends     # displays dependencies required for the package
pkgnames    # displays all packages installed on system
show        # displays info about package
showpkg     # displays info about specified package
stats       # displays package stats for the system
unmet       # displays any unmet dependencies for the installed packages
```

### apt-get

[AptGet/Howto](https://help.ubuntu.com/community/AptGet/Howto)

```bash
# check for broken dependencies
apt-get check
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done

apt-get clean                     # clean up database and any tempory download files in cache
du -sh /var/cache/apt/archives/   # how much space cache uses


# retrieve updated information about pacakges in the repo
apt-get update
```

### apt

Installs and removes pacakges. Has the following options:

```bash
apt action program

# options
autoremove      # rm unneeded packages installed as dependency of another package
autoremove      # rm package and dependencies
build-dep       # installs dependencies for <package>
full-upgrade    # same as upgrade but removes any installed packages req'd to upgrade entire system
install         # installs new package from repository
list            # displays currently installed packages
policy          # Shows available version and repository
purge           # rms specified application and any config or data files
reinstall       # reinstalls existing package from the repo
remove          # rm application but keep config and data files
satisfy         # resolve software dependencies in the installed packages
search          # search for a specific package in the repo
show            # displays info about the package
update          # downloads package info from all repositories in /etc/apt/sources.list.d/sources.list
upgrade         # upgrades all installed packages

# remove a package and its dependencies
apt remove <package>
apt purge  <package>
```

## Red Hat repository tools

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

## Launchpad PPA repos

These are cutting-edge packages that developers sometimes update daily.

Launchpad Personal Package Archive (PPA) updates the `sources.list.d` file. Add repos with this command:

```bash
add-apt-repository ppa:<repo-name>
```

## Application containers

- Containers bundle all files required for an application, including dependencies, into one distro package (the container)
- Each app has exactly the correct dependencies and versions
  - Caveat: dependencies shared among multiple applications are duplicated for each application

### snap containers

Snaps are distro-agnostc software packages:
- Created and maintained by Canonical
- `snap` is an application container format 
- `snapd` app manages the snap packages
- has CLI tool
- Distros just need to be snap-compliant
- Removes differences with distro package managers like `apt` and `yum`
- Stored in `/var/snap/<snap-name>`
- Often not as configurable as manually installed applications

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

## Search for installed software

```bash
# list all installed software
dpkg --list
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                 Version                                 Architecture Description
+++-====================================-=======================================-============-================================================================================
ii  adduser                              3.137ubuntu1                            all          add and remove users and groups
ii  amd64-microcode                      3.20231019.1ubuntu2.1                   amd64        Processor microcode firmware for AMD CPUs
ii  anacron                              2.3-39ubuntu2                           amd64        cron-like program that doesn't go by time

# remove package
apt-get remove <package>
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

## Build from source

- [CompilingEasyHowTo](https://help.ubuntu.com/community/CompilingEasyHowTo)

### Getting the software

```bash
# libraries, tools, compilers to compile C and Cpp
apt install build-essential

# get tarball (or archive) with wget
wget -v https://nmap.org/dist/nmap-7.95.tar.bz2

# decompress and extract files from archive
tar -jxvf nmap-7.95.tar.bz2
```

### Read the INSTALL file (./configure, make, install)

- [The magic behind configure, make, and make install](https://thoughtbot.com/blog/the-magic-behind-configure-make-make-install)

The `INSTALL` file contains information that helps you install the software:

```bash
Ideally, you should be able to just type:

./configure
make
make install

For far more in-depth compilation, installation, and removal notes,
read the Nmap Install Guide at https://nmap.org/book/install.html.
```

- `./configure`: Script that checks your OS version, chip, etc. Might accept params so you can create a custom installation. For example, if you do not need GUI support.
- `make`: Compiles the software with the `gcc` compiler. 
- `make install`: Installs the compiled software on your system. Usually requires root perms--`sudo make install`.

### Locating newly installed software

Important binary locations:
- `/bin`: Key parts of the OS
- `/usr/bin`: Less critical utilities
- `/usr/local/bin`: Software that the user installed themselves

If you install a newer version of software on your system, it executes the first one in its $PATH:

```bash
echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```
Use `locate` to find where the new software is:

```bash
sudo updatedb
locate nmap

```

## Install a tarball with desktop entry

- [Desktop Entry Specification](https://specifications.freedesktop.org/desktop-entry-spec/latest/)
- [Guide to Desktop Entry file](https://www.baeldung.com/linux/desktop-entry-files)

Install a tarball and create a link available in either your application directory or desktop. These steps install the waterfox web browser:

```bash
# 1. Download
wget https://cdn1.waterfox.net/waterfox/releases/6.5.2/Linux_x86_64/waterfox-6.5.2.tar.bz2

# 2. Move to /opt (or whereever you want)
mv waterfox-6.5.2.tar.bz2 /opt/

# 3. Extract tar file
tar -xjf waterfox-6.5.2.tar.bz2

# 4. Make yourself owner of extracted dir
chown -R $USER /opt/waterfox

# 5. Create desktop entry in ~/.local/share/applications
vim ~/.local/share/applications/waterfox.desktop

# 6. Add this content
[Desktop Entry]
Name=Waterfox
Exec=/opt/waterfox/waterfox %u
Terminal=false
Icon=/opt/waterfox/browser/chrome/icons/default/default128.png
Type=Application
Categories=Application;Network;X-Developer;

# 7. Make the desktop entry executable:
chmod +x ~/.local/share/applications/waterfox.desktop

# 8. Remove the tarball
rm /opt/waterfox-6.5.2.tar.bz2

# 9. (Optional) Move to Desktop for icon
mv /opt/waterfox-6.5.2.tar.bz2 ~/Desktop
```