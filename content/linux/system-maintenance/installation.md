+++
title = 'Installation'
date = '2025-09-07T18:47:53-04:00'
weight = 10
draft = false
+++



## Versions

Ubuntu releases a new server version every 6 months. Long Term Support (LTS) versions receive 5 years of support. Non-LTS (interim) releases are supported for 9 months.

## Installation process

Each section below corresponds to a screen in the Ubuntu Server installation wizard.

### GNU GRUB

**Test memory** runs a diagnostic program that detects errors in your RAM modules. Run it once a year.

### Guided storage configuration

**Custom storage layout** lets you create your own partitioning scheme. Consider creating a separate partition for:

- `/var`: stores application log files
- `/home`: isolates user files and configuration from system upgrades

The default configuration creates partitions for the root directory (`/`) and `/boot`.

### Profile setup

Create a user account for post-installation configuration. This account is granted `sudo` privileges by default.