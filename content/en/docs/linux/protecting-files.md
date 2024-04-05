---
title: "Protecting files"
weight: 110
description: >
  "Protecting files"
---

A backup is sometimes called an _archive_, and it is a copy of data that can be restored sometime in the future if the data becomes corrupted. You need to consider the following:
- Backup type
- Compression methods
- Utilities that will help the most

## Understanding backup types

_System image_
: A clone, a copy of the OS binaries, config files, and whatever you need to boot.

_Full_
: Copy of all data, ignoring its modification date. Quickly restores system data, but takes a long time to create the backup.

_Incremental_
: Copy of data that has been modified since the last backup operation, by comparing timestamps. This method is quick, but might take a long time to actually restore.

_Differential_
: Copy of all data that changed since last full backup. Good balance between full and incremental backup.

_Snapshot_
: Hybrid approach - a full (usually read-only) copy of data is made to backup media. Then pointers (ex: hard links) are employed to create a reference table linking the backup data with the original data. During next backup, only modified files are copied to backup media, and the pointer reference table is copied and updated.
 
  You can go back to any point in time (restore point) and restore the data from there. Very efficient and takes less space and processing power.

  `rsync` uses the snapshot approach.

_Snapshot clone_
: After a snapshot is created, it is cloned. Useful in high I/O environments. It is modifiable and mountable, so you can use it as disaster recovery.

## Compression methods

