---
title: "Installing Ubuntu Server"
linkTitle: "Installation"
weight: 1
# description:
---


## Versions

- New Ubuntu server versions are released every 6 months
- LTS (Long Term Support) versions get 5 years of support
- Non-LTS (interim release) versions are supported for 9 months

## Installation process

Some notes about Ubuntu Server installation. Each subsection has the same name as its related screen in the installation process.

### GNU GRUB

**Test memory** runs a program that detects errors in your device's RAM modules. You should run this one time a year.

### Guided storage configuration

**Custom storage layout** lets you create your own partitioning scheme. You might want to create a separate partition for one of the following directories:
- `/var`: Stores lots of application log files
- `/home`: Protects your user files and configs from server upgrades or issues

The default configuration creates a partition for the root directory (`/`) and `/boot`.

### Profile setup

Create user account for post-installation configuration:
- Has sudo privileges