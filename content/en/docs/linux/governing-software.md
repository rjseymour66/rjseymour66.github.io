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