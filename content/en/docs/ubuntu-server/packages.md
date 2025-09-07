---
title: "xSoftware Packages"
linkTitle: "Packages"
weight: 20
# description:
---

Linux uses software repositories to distribute software:
- Available as mirrors, which are copies of the repos in different geographical locations so you can download software from whichever is closest
- Some packages have dependencies, and Linux provides software to help you manage dependencies
- There is a maintainer or group of maintainers that packages and ships new versions to repositories for approval, which are eventually distributed to mirrors
- New package versions are usually because of security updates or new features
  - Updates are usually security patches rather than new features
  - If there are major feature changes, then the new version is held until the upcoming major release
- New features are called **Stable Release Updates** and go through rigorous approval before they are available in default repositories

## Debian packages

[Ubuntu Packages Search](https://packages.ubuntu.com/)

Ubuntu uses Debian packages (`.deb`) because Ubuntu is forked from Debian and uses the same installation commands
- Kernel, system pacakges, libraries, and security updates are all Debian packages
- Other apps - such as Apache - use .deb files too, and their dependency versions might conflict with those used by the system
  - Maintainers go through great pains to ensure there are no conflicts
- New major versions are usually not available until the next distro version release
- **Universal packages** are a new and intended to be a single package that multiple distro versions can recognize
  - Dev creates a single package, instead of one package per distro, and users download that one package, irregardless of their distro

### apt

Advanced Package Tool is a suite of tools that lets you install and manage packages over the command line.
- `apt get` is a legacy command, use `apt` moving forward
- `apt install` process:
  1. Checks package dependencies
  2. List additional packages to install
  3. List suggested  packages - don't install these bc they're not necessary, take space, and increase attack surface of server
  4. Lists new packages to install again
  5. Sumarizes number of packages and disk space they take up
- some packages automatically starts daemons and configures to start at boot
- `apt update` checks local mirror to see if packages were added or removed and updates your local index


`apt-get` and `apt-cache` have a lot of low-level commands that were not commonly used. `apt` consists of the most widely used features of `apt-get` and `apt-cache`. `apt` can install new packages or the kernel, but `apt-get` cannot

`apt` tool summary:
- `apt-cache`: 
- `apt-get`: installs, updates, and removes packages
- `apt`: front end script that can call either `apt-cache` or `apt-get`
- `sudo aptitude`: opens an `apt` GUI in the terminal
- actions logged in `/var/log/dpkg.log`

```bash
apt <action> <package> [<package> <package> ...]

# --- actions --- #
autoremove      # rm unneeded packages installed as dependency of another package
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

# accept required dependency packages to install
apt install <package> -y

# install package and suggested dependencies
apt install <package> --install-suggests

# remove a package and its config (typically in /etc)
apt remove --purge <package>

# get details about a package
apt-cache show <package>
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

### apt-cache

Provides info for the package database:

```bash
depends     # displays dependencies required for the package
pkgnames    # displays all packages installed on system
show        # displays info about package
showpkg     # displays info about specified package
stats       # displays package stats for the system
unmet       # displays any unmet dependencies for the installed packages

# show package details before installing
apt-cache show libapache2-mod-php
Package: libapache2-mod-php
Architecture: all
Version: 2:8.3+93ubuntu2
...
```

### dpkg

Install, remove, and build packages, but cannot download and install packages or their dependencies:
- Older version of `apt`

```bash
dpkg --configure -a             # complete config for packages
dpkg -l                         # list all installed software
dpkg --list
dpkg -L <package>               # list all file locations for package
dpkg -i <file.deb>              # install a deb file
dpkg -r <package-name>          # uninstall a package

dpkg -s openssh-server          # get installed version and update status
Package: openssh-server
Status: install ok installed
Priority: optional

dpkg --get-selections > <file>  # export installed packages to <file>
```

### Adding repositories

The default repos listed in `/etc/apt/sources.list.d/ubuntu.sources` might not contain packages that you need:
- `apt` searches the `/etc/apt/sources.list.d/` directory and reads any file that ends in `.list`
  - Do not edit the `sources.list.d` or `ubuntu.sources` file directly
  - Add one repo per file so you can just delete the file rather than manage complicated `.list` files
  - Might require that you install GNU Privacy Guard (GnuPG) key for a new repo that verifies you are installing signed packages
- Only add repos that you trust
- If repo packages haven't been updated in a while, consider deleting the repo bc it might not be actively maintained.
- `deb` points are binary
- `deb-src` points to source files
- Each software line has one or more components:
  - `main`: Officially supported software, Ubuntu devs have access to src files and can fix bugs
  - `restricted`: Supported but questionable license
  - `universe`: Community supported, no Canonical supported
  - `multiverse`: Not free or supported, use at own risk

```bash
# update local cache after adding repo
apt update

# --- Ubuntu 22 --- #
cat /etc/apt/sources.list
# deb cdrom:[Ubuntu 18.04 _Bionic_ - Build amd64 LIVE Binary 20180608-09:38]/ bionic main

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://us.archive.ubuntu.com/ubuntu/ jammy main restricted
# deb-src http://us.archive.ubuntu.com/ubuntu/ bionic main restricted
...


# --- Ubuntu 24 --- #
cat /etc/apt/sources.list
# Ubuntu sources have moved to /etc/apt/sources.list.d/ubuntu.sources

cat /etc/apt/sources.list.d/ubuntu.sources
Types: deb
URIs: http://us.archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
...


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
### Backup/Restore

Know how to track which packages you have installed if you need to rebuild your server:
- Use `dkpg` command and `apt-get` commands
- Install `dselect`, a supplemental Debian package management tool

```bash
dpkg --get-selections > packages.list     # 1. dump packages to standard text file
apt update                                # 2. Update index
apt install dselect                       # 3. Install dselect and refresh its package index
dselect update

dpkg --set-selections < packages.list     # 4. Install packages - only installs packages that are not installed
apt-get dselect-upgrade
```

### Clean up unused packages

When you remove packages, the dependencies are not always removed too:
- When you install a new package, `apt` lists packages that are no longer required 
- Do not automatically delete kernel pacakges:
  - Contain `linux-image` in the name
  - Maybe wait a week before running `apt autoremove`

```bash
apt autoremove                                # system automatically removes packages
apt remove --purge <package> [<package>...]   # manually remove packages
```

### Auto-upgrade script

Place this script in `/etc/cron.daily/`. `apt` runs as root, so use `sudo` to manually run, or use `cron`, which uses root privileges:

```bash
#!/bin/bash
# Automate regular software updates

apt update
apt upgrade -y
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


## Snap packages

Snap packages is a **universal package** type, which is a single package format that any distro can install
- Universal packages have their dependencies built in, so there shouldn't be conflicts
- Example universal packages include Flatpak, AppImage, and Snap
  - Flatpak and AppImage only work for desktop GUI systems
  - Snap works for both GUI and server installations
- Snap packages are called _snaps_ for short
- Snaps have no dependencies on your system packages
- Snaps are larger than regular Debian packages because they include all dependencies
- Snap packages are just now catching on, we usually still install Debian packages

### snap

`snap` installs Snap pacakges (snaps):
- Typically newer versions than `apt` repos

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

snap install nmap         # install snap package
snap refresh <package>    # update snap
snap remove nmap          # remove snap pacakge
snap disable <package>    # disable
snap enable <package>     # enable
```

## Personal Package Archives

[PPAs for Ubuntu](https://launchpad.net/ubuntu/+ppas) official site

Each PPA is a mini-repo where a vendor might make their source code available:
- Added to server with `apt-add-repository:<username>/<PPA-name>-1.0`
- Only use when you absolutely must
  - If an app version is not available for your distro version, you can use PPA to get the version you need. Otherwise, you might need to compile the app from source and assume responsibility for security patches
- Can manage with `apt`
- Security risk because software is not audited
- Only install if regularly updated

When you install a PPA, the `apt-add-repository` command adds a repo file to your `/etc/atp/sources.list.d/` directory and installs the GPG key.
- To uninstall, delete the repo file in `/etc/atp/sources.list.d/`

```bash
# don't install this, just an example
sudo add-apt-repository ppa:sergey-dryabzhinsky/php55
sudo apt update
```

## Commands

### which

Displays the path to an installed binary
- No output if binary is not found

```bash
which bash
/usr/bin/bash

which nmap
/snap/bin/nmap
```

### whereis

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