---
title: "Containers"
weight: 130
description: >
  Working with Containers in Go.
---

Containers package your application and all the required dependencies using a standard image format, and they run the application in isolation from other processes running on the same system.

A Dockerfile is a recipe for how to create an image. Go is well-suited for containerization because it creates a single binary that does not require additional runtimes or dependencies.

## Build options

CGO_ENABLED=0
: Enables statically linked binaries to make the application more portable. You can use the go binary with images that do not support shared libraries.

GOOS=linux
: Containers run on Linux, so set this to enable repeatable bulds even with building the application on a different platform.

-ldflags="-s-w"
: `-ldflags="-s -w"` lets you specify additional linker options that go build uses at the link stage. `-s -w` strips the binary of debugging symbols to decrease its size.
: To see all linker options, use `go tool link`.

`-tags=containers`
: Build files that have the `// +build containers` directive at the top.

Execute `go build` with these options and compare the binaries:

```shell
## optimized for containers
$ CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -tags=containers
$ ls -lh pomo
-rwxrwxr-x 1 ryanseymour ryanseymour 7.2M Jan 12 23:59 pomo
$ file pomo
pomo: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, stripped

## standard build
$ go build
$ ls -lh pomo
-rwxrwxr-x 1 ryanseymour ryanseymour 13M Jan 13 00:01 pomo
$ file pomo
pomo: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=d921fee20837a42fbe52f957a9fe6643393711cf, with debug_info, not stripped

```

## Dockerfile

After you build a binary that is optimized for containers, create a Dockerfile to build the image. The following Dockerfile creates an image with a regular user `pomo`, and copies the binary into the `/app` directory:

```Dockerfile
FROM alpine:latest 
RUN mkdir /app && adduser -h /app -D pomo 
WORKDIR /app 
COPY --chown=pomo /pomo/pomo .
CMD ["/app/pomo"]
```

Next, build the image:
```shell
$ docker build -t pomo/pomo:latest -f containers/Dockerfile .
Sending build context to Docker daemon  62.86MB
Step 1/5 : FROM alpine:latest
latest: Pulling from library/alpine
8921db27df28: Pull complete 
Digest: sha256:f271e74b17ced29b915d351685fd4644785c6d1559dd1f2d4189a5e851ef753a
Status: Downloaded newer image for alpine:latest
 ---> 042a816809aa
Step 2/5 : RUN mkdir /app && adduser -h /app -D pomo
 ---> Running in aa2c45fe24e7
Removing intermediate container aa2c45fe24e7
 ---> 8315b0cd3500
Step 3/5 : WORKDIR /app
 ---> Running in fd9e0b3fcedc
Removing intermediate container fd9e0b3fcedc
 ---> 1c29d6e70641
Step 4/5 : COPY --chown=pomo /pomo/pomo .
 ---> f9cd8113caa2
Step 5/5 : CMD ["/app/pomo"]
 ---> Running in 0c3a50e3b4ed
Removing intermediate container 0c3a50e3b4ed
 ---> f5bbd680ac85
Successfully built f5bbd680ac85
Successfully tagged pomo/pomo:latest


$ docker images
REPOSITORY                 TAG       IMAGE ID       CREATED          SIZE
pomo/pomo                  latest    f5bbd680ac85   22 seconds ago   14.5MB
...


```
## Multistage builds

Multistage builds create smaller image sizing. The following Dockerfile uses Go's official image. With this image, you don't have to compile the binary before creating the image:

```Dockerfile
FROM golang:1.19 AS builder
RUN mkdir /distributing 
WORKDIR /distributing 
COPY notify/ notify/
COPY pomo/ pomo/
WORKDIR /distributing/pomo
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -tags=containers

FROM alpine:latest 
RUN mkdir /app && adduser -h /app -D pomo 
WORKDIR /app 
COPY --chown=pomo /pomo/pomo .
CMD ["/app/pomo"]
```


You can also build statically linked binaries and share them with a multistage build that does not include an operating system image or users:

```shell
#Dockerfile.scratch

FROM golang:1.19 AS builder
RUN mkdir /distributing 
WORKDIR /distributing 
COPY notify/ notify/
COPY pomo/ pomo/
WORKDIR /distributing/pomo
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -tags=containers

FROM scratch
WORKDIR /
COPY --from=builder /distributing/pomo/pomo .
CMD ["/pomo"]
```