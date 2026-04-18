+++
title = 'Files and directories'
date = '2025-09-07T18:57:14-04:00'
weight = 50
draft = false
+++


## Streams

### STDIN, STDOUT, STDERR

Every program is a process, and every process has three distinct file descriptors:

| Stream   | File descriptor | Description                                |
| :------- | :-------------: | :----------------------------------------- |
| `stdin`  | 0               | The default source of input to a program   |
| `stdout` | 1               | The default destination for program output |
| `stderr` | 2               | The default destination for error messages |


### Redirection and piping

Redirection lets you change where a program reads input from and sends output to, without modifying the program itself:

| Operator     | Description                                     | Example                                            |
| :----------- | :---------------------------------------------- | :------------------------------------------------- |
| `>`          | Redirects STDOUT to a file                      | `ls -la > listing.out`                             |
| `<`          | Sends a file to a program as STDIN              | `program < input.txt`                              |
| `2>`         | Redirects STDERR to a file                      | `cp -r /etc/a /etc/b 2> err.msg`                   |
| `2>&1`       | Redirects STDERR to the same location as STDOUT | `program > output.out 2>&1`                        |
| `&>`         | Shorthand to redirect both STDOUT and STDERR    | `program &> output.out`                            |
| `>` and `2>` | Redirect STDOUT and STDERR to separate files    | `find / -name 'syslog' > stdout.txt 2> stderr.txt` |

#### Redirect STDIN and STDOUT

The following command sends `input.txt` to `program` as STDIN and redirects STDOUT to `output.out`:

```bash
program < input.txt > output.out
```

#### Redirect to separate files

To separate STDOUT and STDERR into different files, combine both operators:

```bash
program < input.txt > output.out 2> err.msgs
```

#### Combine STDOUT and STDERR

To send STDERR to the same destination as STDOUT, append `2>&1`. The `&>` shorthand is equivalent:

```bash
program < input.txt > output.out 2>&1
program < input.txt &> output.out
```

#### Discard STDOUT

To discard STDOUT entirely, redirect it to `/dev/null`:

```bash
program < input.txt > /dev/null
```

#### Split output with tee

The `tee` command sends output to both STDOUT and a file. Use the `-a` flag to append instead of overwrite:

```bash
program < input.txt | tee results.out
program < input.txt | tee -a results.out
```

#### Append output

To append STDOUT to a file, use `>>`. To append both STDOUT and STDERR, use `&>>`:

```bash
program < input.txt >> appended.file
program < input.txt &>> appended.file
```

### Running commands in the background

#### Run a command in the background

Append `&` to any command to run it in the background. Redirect both STDOUT and STDERR to a file so output is captured and not lost:

```bash
ping 10.20.30.40 &> ping.log &
```

#### Bring a background job to the foreground

Run `jobs` to list background tasks, then run `fg` with the job number to bring it forward:

```bash
ping 192.168.10.56 &> ping.log &
[1] 7452
jobs
[1]+  Running                 ping 192.168.10.56 &> ping.log &
fg 1
ping 192.168.10.56 &> ping.log
^C
```

#### Suspend and resume a job

Stop a running foreground job with `Ctrl+z`. Send it to the background with `bg`, then bring it forward again with `fg`:

```bash
fg 1
ping 192.168.10.56 &> ping.log
^Z
[1]+  Stopped                 ping 192.168.10.56 &> ping.log
bg 1
[1]+ ping 192.168.10.56 &> ping.log &
fg 1
ping 192.168.10.56 &> ping.log
^C
```

## Links

Linux supports two types of links: hard links and symbolic links. Both let you access a file from more than one location, but they work differently at the filesystem level.

An inode is a filesystem object that stores metadata about a file or directory, including the owner, permissions, last modified date, and file type. Understanding inodes is key to understanding how both link types work.

### Hard link

A hard link is a filename that points to the same inode as another file. Both filenames reference the same data on disk. Deleting one does not remove the data as long as the other filename still exists.

Hard links have the following constraints:
- You can only create hard links to files, not directories
- Both filenames must exist on the same filesystem
- Moving either filename anywhere on the filesystem does not break the link

#### File backup

Hard links work well as file backups when disk space is limited. If one filename is deleted, the data remains accessible through the other.
  
#### Syntax

```bash
ln ORIGINAL LINKED-FILE
unlink LINKED-FILE
```

#### Example

1. Create the original file and a separate file for comparison:

   ```bash
   touch original-file.txt
   touch single-file.txt
   ```

2. Create a hard link:

   ```bash
   ln original-file.txt hard-link-file.txt
   ```

3. View the inodes. Notice that `hard-link-file.txt` and `original-file.txt` share inode `4868127`:

   ```bash
   ls -iog
   total 0
   4868127 -rw-rw-r-- 2 0 Mar 17 09:32 hard-link-file.txt
   4868127 -rw-rw-r-- 2 0 Mar 17 09:32 original-file.txt
   4868128 -rw-rw-r-- 1 0 Mar 17 09:33 single-file.txt
   ```

4. Remove the hard link:

   ```bash
   unlink hard-link-file.txt
   ```

5. The original file remains and the link count drops to `1`:

   ```bash
   ls -iog
   total 0
   4868127 -rw-rw-r-- 1 0 Mar 17 09:32 original-file.txt
   4868128 -rw-rw-r-- 1 0 Mar 17 09:33 single-file.txt
   ```


### Symbolic (soft) link

A symbolic link is a pointer to another file's path, not a copy of its data. Because it references a path rather than an inode, it has a unique inode and can point to files on other filesystems.

Symbolic links have the following characteristics:
- Linked files have different inodes
- You can link to files or directories
- You can link across filesystems and disk volumes
- Moving or deleting the original file breaks the link, but does not delete the symbolic link itself

{{< admonition "Dangling symlinks" warning >}}
If the original file is deleted and a malicious file is placed at the same path, the symbolic link will point to it. Audit dangling symlinks regularly.
{{< /admonition >}}

#### Apache virtual hosts

For Apache, you can disable a site by removing the symlink in `/etc/apache2/sites-enabled/` without deleting the config in `/etc/apache2/sites-available/`. Re-add the link to reactivate the site.

#### Syntax

```bash
ln -s ORIGINAL LINKED-FILE
unlink LINKED-FILE
```

#### Example

1. View the current directory contents:

   ```bash
   ls -iog
   total 0
   4868127 -rw-rw-r-- 1 0 Mar 17 09:32 original-file.txt
   ```

2. Create a symbolic link:

   ```bash
   ln -s original-file.txt soft-link-file.txt
   ```

3. View the inodes. Notice that `soft-link-file.txt` has a different inode from `original-file.txt`:

   ```bash
   ls -iog
   total 0
   4868127 -rw-rw-r-- 1  0 Mar 17 09:32 original-file.txt
   4868128 lrwxrwxrwx 1 17 Mar 17 09:45 soft-link-file.txt -> original-file.txt
   ```

4. Remove the symbolic link:

   ```bash
   unlink soft-link-file.txt
   ```

5. The original file remains:

   ```bash
   ls -iog
   total 0
   4868127 -rw-rw-r-- 1 0 Mar 17 09:32 original-file.txt
   ```