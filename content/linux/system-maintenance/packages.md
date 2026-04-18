+++
title = 'Software packages'
linkTitle = 'Packages'
date = '2025-09-07T18:48:26-04:00'
weight = 20
draft = false
+++


Linux distributes software through repositories hosted as mirrors, which are copies spread across geographic locations so you can download from the nearest one. Package managers handle dependencies automatically. Maintainers package new software versions and submit them for approval before they are distributed to mirrors.

Most package updates address security vulnerabilities rather than introduce new features. Major feature changes typically wait for the next major distribution release. In Ubuntu, fixes and patches that are safe to backport to a stable release go through an approval process called **Stable Release Updates (SRUs)**.

## Debian packages

[Ubuntu Packages Search](https://packages.ubuntu.com/)

Ubuntu uses Debian packages (`.deb`) because Ubuntu is forked from Debian and uses the same installation commands. The kernel, system packages, libraries, and security updates are all Debian packages. Third-party applications such as Apache also use `.deb` files, though their dependency versions can occasionally conflict with system packages. Maintainers work to prevent these conflicts, but they can occur.

New major versions of packages are typically not available until the next distribution release. **Universal packages** are a newer format designed as a single package that any distribution can install, regardless of version. Rather than maintaining one package per distribution, a developer publishes one package that all users download.

### apt

Advanced Package Tool (apt) is a suite of tools for installing and managing packages from the command line. `apt-get` is a legacy command. Prefer `apt` for all package management going forward. `apt` consolidates the most commonly used features of `apt-get` and `apt-cache`, and unlike `apt-get`, it can install new kernel packages. All actions are logged in `/var/log/dpkg.log`.

`apt update` checks the local mirror index for added or removed packages and updates your local cache. Some packages automatically configure a daemon to start at boot when installed.

Debian package management uses several related command-line tools:

| Tool        | Description                                          |
| :---------- | :--------------------------------------------------- |
| `apt-cache` | Queries the local package index                      |
| `apt-get`   | Installs, updates, and removes packages              |
| `apt`       | Front-end script that calls `apt-cache` or `apt-get` |
| `aptitude`  | Terminal GUI for `apt`                               |

### apt install

When you run `apt install`, it works through the following steps:

1. Checks package dependencies.
2. Lists additional packages required to install.
3. Lists suggested packages. Do not install these. They are unnecessary, consume disk space, and increase the attack surface of the server.
4. Summarizes the number of packages and disk space required.

### apt commands

`apt` accepts an action and one or more package names:

```bash
apt <action> <package> [<package> ...]

# --- actions --- #
autoremove      # remove unneeded packages installed as dependencies
build-dep       # install dependencies for <package>
full-upgrade    # upgrade all packages, removing installed packages if required
install         # install a new package from the repository
list            # list currently installed packages
policy          # show available versions and source repository
purge           # remove a package and its configuration files
reinstall       # reinstall an existing package from the repository
remove          # remove a package but keep its configuration files
satisfy         # resolve dependency conflicts in installed packages
search          # search for a package in the repository
show            # show details about a package
update          # download package information from all configured repositories
upgrade         # upgrade all installed packages

# accept required dependency packages without prompting
apt install <package> -y

# install a package and its suggested dependencies
apt install <package> --install-suggests

# remove a package and its configuration files (typically in /etc)
apt remove --purge <package>

# show details about a package before installing
apt-cache show <package>
```
### apt-get

[AptGet/Howto](https://help.ubuntu.com/community/AptGet/Howto)

`apt-get` provides lower-level package management operations. Prefer `apt` for day-to-day use:

```bash
# check for broken dependencies
apt-get check
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done

apt-get clean                     # remove cached package files from /var/cache/apt/archives/
du -sh /var/cache/apt/archives/   # show how much space the cache is using

# download updated package information from all configured repositories
apt-get update
```

### apt-cache

`apt-cache` queries the local package index without modifying the system:

```bash
depends     # list dependencies required by a package
pkgnames    # list all packages known to the package index
show        # show details about a package (human-readable)
showpkg     # show low-level details about a package
stats       # show package statistics for the system
unmet       # list unmet dependencies in installed packages

# show package details before installing
apt-cache show libapache2-mod-php
Package: libapache2-mod-php
Architecture: all
Version: 2:8.3+93ubuntu2
...
```

### dpkg

`dpkg` is the low-level package manager that `apt` is built on. Reach for it when working with local `.deb` files or diagnosing package state. It does not resolve or download dependencies, so prefer `apt` for most tasks:

```bash
dpkg --configure -a             # complete configuration for any partially installed packages
dpkg -l                         # list all installed packages
dpkg -L <package>               # list all installed file paths for a package
dpkg -i <file.deb>              # install a local .deb file
dpkg -r <package-name>          # remove a package

dpkg -s openssh-server          # show installed version and status
Package: openssh-server
Status: install ok installed
Priority: optional

dpkg --get-selections > <file>  # export a list of installed packages to <file>
```

### Adding repositories

When the default repositories in `/etc/apt/sources.list.d/ubuntu.sources` do not contain a package you need, you can add a third-party repository. `apt` scans `/etc/apt/sources.list.d/` and reads any file ending in `.list`. Add one repository per file so you can remove it by deleting that file rather than editing a shared list. Some repositories require a GNU Privacy Guard (GnuPG) key to verify that packages are signed. Only add repositories you trust. If a repository has not been updated in a long time, consider removing it.

Each source entry uses either `deb` (binary packages) or `deb-src` (source packages). Entries also specify one or more components:

| Component    | Description                                                        |
| :----------- | :----------------------------------------------------------------- |
| `main`       | Officially supported software with Canonical-maintained source     |
| `restricted` | Supported software with proprietary or restricted licensing        |
| `universe`   | Community-maintained software with no official Canonical support   |
| `multiverse` | Unsupported software that may have licensing or legal restrictions |

After adding a repository, update the local cache:

```bash
apt update
```

To inspect the configured sources on Ubuntu 22 and earlier:

```bash
cat /etc/apt/sources.list
deb http://us.archive.ubuntu.com/ubuntu/ jammy main restricted
...
```

On Ubuntu 24, sources moved to a dedicated file:

```bash
cat /etc/apt/sources.list.d/ubuntu.sources
Types: deb
URIs: http://us.archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
...
```

To list only active (non-commented) repository entries:

```bash
grep -v "#" /etc/apt/sources.list
deb http://gb.archive.ubuntu.com/ubuntu/ jammy main restricted
deb http://gb.archive.ubuntu.com/ubuntu/ jammy-updates main restricted
...
```
### Back up and restore packages

If you need to rebuild a system, you can export your installed package list and restore it on the new machine. This requires `dselect`, a supplemental Debian package management tool.

1. Export the current package list to a file:

   ```bash
   dpkg --get-selections > packages.list
   ```

2. On the new machine, update the package index and install `dselect`:

   ```bash
   apt update
   apt install dselect
   dselect update
   ```

3. Restore the package selections. This installs only packages not already present:

   ```bash
   dpkg --set-selections < packages.list
   apt-get dselect-upgrade
   ```

### Clean up unused packages

When you remove a package, its dependencies are not always removed automatically. `apt` flags these orphaned packages after installation. Avoid automatically removing kernel packages. Any package with `linux-image` in its name is a kernel package. Wait at least a week before running `apt autoremove` to confirm the old kernel is no longer needed:

```bash
apt autoremove                                # remove orphaned dependency packages
apt remove --purge <package> [<package>...]   # remove packages and their configuration files
```

### Auto-upgrade script

Scripts placed in `/etc/cron.daily/` run automatically as root. To run this script manually, use `sudo`:

```bash
#!/bin/bash
# Automate regular software updates

apt update
apt upgrade -y
```

## Red Hat repository tools

Red Hat-based systems originally used `yum` (YellowDog Update Manager). It has been replaced by `dnf`, the modern package manager for querying, installing, and removing packages on Red Hat-based systems. Both `dnf` and `yum` read repository definitions from `/etc/yum.repos.d/`, where each file lists a repository URL and the location of its package metadata.

### dnf

`dnf` accepts an action and a package name:

```bash
dnf <action> <program>
alias               # define an alias pointing to a list of other dnf commands
autoremove          # remove unneeded packages installed as dependencies
check               # examine the local package database and report problems
check-update        # check the repository for updates to a specified package
clean               # remove temporary files cached for repositories
deplist             # deprecated alias for the repoquery command
distro-sync         # downgrade or install packages to sync the system with current repositories
downgrade           # downgrade a package to the version in the repository
group               # manage a set of packages as a single entity
help                # show help
history             # show previous dnf commands
info                # show details about installed and available packages
install             # install the current version of a package
list                # list installed and available packages
makecache           # download metadata for configured repositories
mark                # mark a package as manually installed
module              # manage module packages
provides            # show which package installed a specified file
reinstall           # reinstall a specified package
remove              # remove a package and any packages that depend on it
repoinfo            # show details about configured repositories
repolist            # list currently configured repositories
repoquery           # search configured repositories for a package
repository-packages # run commands on all packages in a repository
search              # search package metadata for keywords
shell               # open an interactive shell for entering multiple dnf commands
swap                # remove one package and install another
updateinfo          # show update advisory messages
upgrade             # install the latest version of specified packages, or all packages
upgrade-minimal     # install only versions that provide a bug fix or security fix
```


## Snap packages

Snap is a **universal package** format that any distribution can install. Unlike Debian packages, snaps bundle all their dependencies, which eliminates version conflicts but results in larger package sizes. Flatpak and AppImage are similar universal formats but work only on desktop systems. Snap works on both desktop and server installations. For most server use cases, Debian packages remain the standard choice.

### snap

`snap` installs and manages snap packages. Snaps are typically newer versions than those available through `apt`.

#### snap version

Show the installed snapd version:

```bash
snap version
snap    2.61.3+22.04
snapd   2.61.3+22.04
series  16
ubuntu  22.04
kernel  5.15.0-91-generic
```

#### snap list

List all installed snap packages:

```bash
snap list
Name                       Version           Rev    Tracking         Publisher      Notes
bare                       1.0               5      latest/stable    canonical✓     base
chromium                   123.0.6312.86     2805   latest/stable    canonical✓     -
core                       16-2.61.2         16928  latest/stable    canonical✓     core
...
```

#### snap find

Search for a package by name:

```bash
snap find cups
Name                              Version                Publisher             Notes  Summary
cups                              2.4.7-8                openprinting✓         -      The CUPS Snap - The Printing Stack for Linux
...
```

#### Manage snap packages

Install, update, and remove snap packages:

```bash
snap install nmap         # install a snap package
snap refresh <package>    # update a snap package
snap remove nmap          # remove a snap package
snap disable <package>    # disable a snap package
snap enable <package>     # enable a snap package
```

## Personal Package Archives

[PPAs for Ubuntu](https://launchpad.net/ubuntu/+ppas) official site

A Personal Package Archive (PPA) is a small repository where a developer publishes packages outside the official Ubuntu repositories. PPAs are managed with `apt` but carry more risk than official packages because the software is not audited. Only add a PPA when a package or version you need is not available through official repositories, and only if the PPA is actively maintained.

When you add a PPA, `add-apt-repository` creates a repository file in `/etc/apt/sources.list.d/` and installs the GPG key. To remove a PPA, delete that file.

Do not run this command. It is shown as an example only:

```bash
sudo add-apt-repository ppa:sergey-dryabzhinsky/php55
sudo apt update
```

## Commands

### which

`which` displays the full path to an installed binary. It produces no output if the binary is not found:

```bash
which bash
/usr/bin/bash

which nmap
/snap/bin/nmap
```

### whereis

`whereis` locates the binary, source, and manual page files for a command:

```bash
whereis go
go: /usr/local/go /usr/local/go/bin/go

whereis aws
aws: /usr/local/bin/aws

whereis tar
tar: /usr/bin/tar /usr/share/man/man1/tar.1.gz
```

## Build from source

- [CompilingEasyHowTo](https://help.ubuntu.com/community/CompilingEasyHowTo)

Building from source gives you the latest version of software when a package is not available in any repository. It requires a C/C++ build toolchain and the source archive.

### Getting the software

Before building from source, install the required build tools and download the source archive.

1. Install the C/C++ build toolchain:

   ```bash
   apt install build-essential
   ```

2. Download the source archive:

   ```bash
   wget -v https://nmap.org/dist/nmap-7.95.tar.bz2
   ```

3. Extract the archive:

   ```bash
   tar -jxvf nmap-7.95.tar.bz2
   ```

### Read the INSTALL file

- [The magic behind configure, make, and make install](https://thoughtbot.com/blog/the-magic-behind-configure-make-make-install)

The `INSTALL` file in the source directory documents how to build the software. Most projects follow the same three-step pattern:

```bash
./configure
make
sudo make install
```

- `./configure`: Checks your system (OS version, CPU architecture, and available libraries) and prepares the build for your environment. It may accept flags to customize the installation, for example to disable GUI support.
- `make`: Compiles the software using the `gcc` compiler.
- `sudo make install`: Copies the compiled files to their destination on your system and requires root permissions.

### Locating newly installed software

Linux stores binaries in standard locations. When multiple versions of the same binary exist, the system runs the first one it finds in `$PATH`:

| Path             | Contents                  |
| :--------------- | :------------------------ |
| `/bin`           | Core OS binaries          |
| `/usr/bin`       | Standard system utilities |
| `/usr/local/bin` | User-installed software   |

```bash
echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

To find where a newly installed binary was placed, update the file database and run `locate`:

```bash
sudo updatedb
locate nmap
```

## Install a tarball with desktop entry

- [Desktop Entry Specification](https://specifications.freedesktop.org/desktop-entry-spec/latest/)
- [Guide to Desktop Entry file](https://www.baeldung.com/linux/desktop-entry-files)

This procedure installs the Waterfox browser from a tarball and registers it as a desktop application.

1. Download the archive:

   ```bash
   wget https://cdn1.waterfox.net/waterfox/releases/6.5.2/Linux_x86_64/waterfox-6.5.2.tar.bz2
   ```

2. Move the archive to `/opt`:

   ```bash
   mv waterfox-6.5.2.tar.bz2 /opt/
   ```

3. Extract the archive:

   ```bash
   tar -xjf waterfox-6.5.2.tar.bz2
   ```

4. Take ownership of the extracted directory:

   ```bash
   chown -R $USER /opt/waterfox
   ```

5. Create a desktop entry file:

   ```bash
   vim ~/.local/share/applications/waterfox.desktop
   ```

6. Add the following content to the file:

   ```bash
   [Desktop Entry]
   Name=Waterfox
   Exec=/opt/waterfox/waterfox %u
   Terminal=false
   Icon=/opt/waterfox/browser/chrome/icons/default/default128.png
   Type=Application
   Categories=Application;Network;X-Developer;
   ```

7. Make the desktop entry executable:

   ```bash
   chmod +x ~/.local/share/applications/waterfox.desktop
   ```

8. Remove the tarball:

   ```bash
   rm /opt/waterfox-6.5.2.tar.bz2
   ```