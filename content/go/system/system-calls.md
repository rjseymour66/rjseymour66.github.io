+++
title = 'System Calls'
date = '2025-11-27T10:07:16-05:00'
weight = 10
draft = false
+++

System calls ("syscalls") are the interface between user-level programs and the operating system kernel. System calls are made through the system call API, a collection of low-level functions provided by the OS kernel to provide user mode processes to request services from the kernel.

Find Linux syscalls with the [searchable syscall table](https://filippo.io/linux-syscall-table/).

## x/sys package

The `syscall` package became so bloated and unwieldly that it is no longer maintained. Now, all syscall functions are available in the `x/sys` and `os` packages.

{{< admonition "Prefer the `os` package" tip >}}
For most tasks, you want to use the `os` package. Use `x/sys` only when you need fine-grained control over system calls for the specific OS.
{{< /admonition >}}

You have to download the `x/sys` package:

```bash
go get -u golang.org/x/sys
```

## os package

### File and directory operations

| Function          | Description                                                                                            |
| :---------------- | :----------------------------------------------------------------------------------------------------- |
| `os.Create()`     | Creates or opens a file for writing.                                                                   |
| `os.Mkdir()`      | Create a directory.                                                                                    |
| `os.MkdirAll()`   | Create a directory and any parent directories. Equivalent to `mkdir -p`.                               |
| `os.Remove()`     | Removes a single file or empty directory.                                                              |
| `os.RemoveAll()`  | Recursively removes a directory and all its contents. Equivalent to `rm -rf`.                          |
| `os.Stat()`       | Get file or directory metadata.                                                                        |
| `os.IsExist()`    | Interprets an error and determines if the file or directory exists. Superceded by `errors.Is`.         |
| `os.IsNotExist()` | Interprets an error and determines if the file or directory does not exist. Superceded by `errors.Is`. |
| `os.Open()`       | Opens a file for reading.                                                                              |
| `os.Truncate()`   | Resizes a file.                                                                                        |
| `os.Getwd()`      | Get the current working directory.                                                                     |
| `os.Chdir()`      | Change the current working directory.                                                                  |
| `os.Args`         | Get the command line arguments.                                                                        |
| `os.Getenv()`     | Get environment variables.                                                                             |
| `os.Setenv()`     | Set environment variables.                                                                             |

### Process and signal operations

| Function                        | Description                                        |
| :------------------------------ | :------------------------------------------------- |
| `os.Getpid()`                   | Get the current process ID.                        |
| `os.Getppid()`                  | Get the parent process ID.                         |
| `os.Getuid()`                   | Get the user ID.                                   |
| `os.Getgid()`                   | Get the group ID.                                  |
| `os.Geteuid()`                  | Get the effective user ID.                         |
| `os.Getugid()`                  | Get the effective group ID.                        |
| `os.StartProcess()`             | Start a new process.                               |
| `os.Exit()`                     | Exit the current process.                          |
| `os.Signal`                     | Represents signals such as `SIGINT` and `SIGTERM`. |
| `os.Notify()`/`signal.Notify()` | Notify on signal reception.                        |