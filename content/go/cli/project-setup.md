+++
title = 'Project Setup'
date = '2025-08-12T23:23:44-04:00'
weight = 10
draft = false
+++


## Project structure

When you build an executable, the convention is to nest `main` and `init` functions within subdirectories of `/cmd`. For example, `/cmd/host/main.go`. You cannot have more than one `main.go` file in a directory.


## CLI tools

Separate your CLI program executable from the client/package logic so your code is flexible. For example, you can use a client package across different programs rather than have it tied to the CLI.

1. Client package.
2. `cmd/` groups executable commands within subdirectories. Each subdirectory in `cmd/` becomes its own binary.
3. CLI tool directory.
4. CLI tool configuration, such as flag parsing.
5. CLI tools entry point (`main`)
6. Daemon, such as an API service.

```bash
hit
├── . . .                       # 1
├── cmd                         # 2
│   ├── hit                     # 3
│   │   ├── config.go           # 4
│   │   ├── config_test.go
│   │   ├── hit.go              # 5
│   │   └── hit_test.go
│   └── hitd                    # 6
└── go.mod

```