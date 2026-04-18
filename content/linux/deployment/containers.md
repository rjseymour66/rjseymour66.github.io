+++
title = 'Containers'
date = '2025-09-07T19:08:06-04:00'
weight = 10
draft = false
+++


A container is an isolated filesystem that shares the host's CPU and kernel. Unlike virtual machines, containers do not require dedicated CPU, RAM, or disk allocations. When you disconnect from a container without leaving a running process, the container stops.

Containers are portable and run on any infrastructure that has a container runtime. They are designed to do one thing, such as run a web server or a browser-based application. Use a container any time you need to run a web application while preserving host resources.

Two common container platforms are Docker and LXD. Docker is general-purpose and runs on all major operating systems. LXD is better suited to Linux hosts. Always audit third-party container images before using them.

## Docker

Docker uses a layered filesystem: every change to a container creates a new layer. You can use existing layers as the base for new containers, which saves disk space.

An *image* is a snapshot of a Linux filesystem configured to perform a specific task. It is a recipe for a container. A *container* is a running instance of an image. Packages installed in a running container do not persist after the container stops.

The *ENTRYPOINT* is the command that runs when a container starts. It is usually a shell script that configures and launches the application.

Docker Hub is a public registry where you can download images that others have already built.


### Install and setup

Follow the [Docker installation instructions](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository) for Ubuntu. After installation, verify the service and add your user to the `docker` group to run commands without `sudo`:

```bash
systemctl status docker                 # verify Docker is running
systemctl enable --now docker           # enable and start Docker if it is not running
usermod -aG docker <username>           # add your user to the docker group
```

### Managing containers

Exiting a container stops it. Detached mode runs a container in the background like a service and keeps running until you explicitly stop it.

Port redirection forwards traffic from a host port to a port inside the container, making a containerized service accessible from a browser. This is not the preferred approach for web servers. Use an `ENTRYPOINT` instead. To access a containerized web server from a browser:

1. Start the container with port forwarding.
2. Attach to the container.
3. Install dependencies.
4. Run the program.
5. Detach without stopping the container.
6. Open the host port in a browser.

Ubuntu containers do not include `sudo` by default, but you connect as root. The apt package index is absent by default, so run `apt update` before installing anything. Containers do not have `systemctl`.

The `COMMAND` column in `docker ps` shows the command that was run when the container was created.


### Commands

Common commands for managing images and containers. The `-i`, `-t`, and `-d` flags control interactive mode, pseudo-TTY, and detached mode respectively:

```bash
docker info                                             # show details about the Docker installation
docker search <keyword>                                 # search Docker Hub for an image
docker pull <image-name>                                # download an image to the local machine
docker images                                           # list downloaded images

docker rmi <image-id>                                   # remove an image
docker rm <container-id>                                # remove a stopped container

docker run <image-name>                                 # create and start a container from an image
docker run -it ubuntu /bin/bash                         # start an Ubuntu container with an interactive shell
docker run -dit <image-name> <command>                  # start a container in detached mode
docker run -dit -p <host-port>:<container-port> <image> <command>   # start with port forwarding
docker run -dit -p 8080:80 ubuntu /bin/bash             # example: forward host port 8080 to container port 80

docker ps                                               # list running containers
docker ps -a                                            # list all containers

docker start <container-id>                             # start a stopped container
docker stop <container-id>                              # stop a running container
docker attach <container-id>                            # attach to a running container
CTRL+P, CTRL+Q                                          # detach without stopping the container

docker commit <container-id> <repo>/<image-name>:<tag>  # create an image from a running container
```

### Dockerfiles

A Dockerfile is a set of instructions for building a Docker image. To create one:

1. Create a directory named after the image you want to build.
2. Inside that directory, create a file named `Dockerfile`.

#### Dockerfile

```dockerfile
FROM ubuntu                                 # base image — downloads from Docker Hub if not present locally
MAINTAINER Ryan <ryan@ryan.com>             # optional: image author
ARG DEBIAN_FRONTEND=noninteractive          # suppress interactive prompts during package installation
RUN apt update && apt dist-upgrade -y       # update packages when building the image
RUN apt install -y apache2 vim-nox          # install required packages
ENTRYPOINT apache2ctl -D FOREGROUND         # command that runs when the container starts
```

Build the image from the directory containing the `Dockerfile`:

```bash
docker build -t <repo>/<image-name>:<tag> .     # build an image from the Dockerfile in the current directory
docker build -t myrepo/apache-server:1.0 .      # example
```

## LXD

LXD (pronounced "Lex-D") is a machine container platform that emulates a virtual machine more closely than Docker. It is a descendant of LXC ("Lex-C", Linux Containers) and builds on top of it by adding snapshots, advanced filesystem support, and live migration.

LXD is developed by Canonical and is available in Ubuntu. It is also distributed as a snap package, so it runs on all distributions. You can access the container's filesystem directly from the host.

Manage containers with the `lxc` command. LXD layer operations use the `lxd` command.

Unlike Docker, LXD does not use layers, so packages installed in a running container persist after the container stops. This also means LXD is slower than Docker to start containers. The default Ubuntu container includes an `ubuntu` user. Run `apt update` each time you log in.

LXD assigns containers their own IP address space. You can configure a bridged network to assign addresses from your DHCP server, which requires a wired interface.


### Setup

Install LXD, add your user to the `lxd` group, and initialize the installation:

```bash
sudo snap install lxd                   # install the LXD snap
usermod -aG lxd <username>              # add your user to the lxd group
lxd init                                # initialize LXD
```

During `lxd init`, set IPv6 to `none` and enable network availability if you want to access LXD over the network:

```bash
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none
Would you like the LXD server to be available over the network? (yes/no) [default=no]: yes
```

### Commands

```bash
lxc launch ubuntu:22.04 <container-name>             # download and start a container
lxc list                                             # list all containers
lxc start <container-name>                           # start a container
lxc stop <container-name>                            # stop a container
lxc delete <container-name>                          # delete a container
lxc image list                                       # list downloaded images
lxc image delete <image-name>                        # delete an image

lxc exec <container-name> bash                       # open a bash shell in a container
lxc exec <container-name> -- su --login <username>   # log in as a specific user
lxc config set <container-name> boot.autostart 1     # start the container automatically at boot
curl <container-ip-addr>                             # verify the container is reachable
```