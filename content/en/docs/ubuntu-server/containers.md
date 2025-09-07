---
title: "xContainers"
# linkTitle: ""
weight: 150
# description:
---

- Containers are isolated filesystems
  - VMs require that we dedicate resources. We provision CPUs, RAM, and disk space - it might not use all allocated resources
  - Containers share CPU, kernel with the host
  - Containers are a collection of namespaces that isolate the container
    - When you disconnect from the container without leaving a running process, the container stops
- Containers are portable and can run on any infrastructure as long as it has a container runtime
- Generally do just one thing (run one app), such as a web server
  - Run a container any time you need to run a web app or need to peserve resources
  - Any app that runs in a browser is good for containers
- Docker is more general purpose than LXD - it runs on all major OSs
- LXD is better for linux hosts
- Always heavily audit 3rd party containers that you use!

## Docker

Docker employs a layered approach to documentation, where every change you make to the container creates a new layer
- You can use these layers as a base for other containers - this saves disk space
- Has an ENTRYPOINT command that executes in the running container
- Docker can be though of as an application container that contains resources to run an app
- Docker has Dockerhub, where you can download containers that others have already created. This saves time
- Image is the closest equivalent that docker has to a VM or hardware image
  - Snapshot wiht linux fs that includes changes that the creator added to perform a specific task
- Application run in Docker is called an ENTRYPOINT
  - Usually a bash script that configures and runs an application
- Image is a recipe for a container, a container is a running image
  - If you install packages in a running container, they do not persist beyond the lifetime of the container


### Install and setup 

Steps to install: https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository

```bash
systemctl status docker                 # verify Docker is running
systemctl enable --now docker           # enable Docker immediately (if not running and not enabled)
usermod -aG docker <username>           # add yourself to docker group so you don't need sudo each time
```

### Managing containers

- When you exit a container, you are stopping it - it no longer runs the program
- `docker ps` - the `COMMAND` is the command that was run when the container was created
- running a container in detached mode is like running a service - the container doesnt stop until you tell it to
- Port redirection lets you send traffic from a port in the container to a port on the host
  - For example, you can access a container's web server instance in a web browser:
    1. Start the container with port forwarding
    2. Attach to container
    3. Install dependencies
    4. Run program
    5. Detach without exiting the container
    6. Go to host port
 - This is not preferred way to start web service container, should use ENTRYPOINT
- sudo is not included by default in ubuntu container, but you attach as root
  - apt repo does not even exist, so run `apt update` before trying to install anything
- containers do not have `systemctl`


```bash
# -i: interactive mode
# -t: psuedo TTY
# -d: detached - run in the bg
docker info                                             # details about current docker installation, images, containers, etc
docker search <keyword>                                 # search for pre-existing image
docker pull <image-name>                                # pull docker image to local machine
docker images                                           # list docker images on your server

docker rmi <image-id>                                   # remove image from your server
docker rm <container-id>                                # remove persistent container

docker run <image-name>                                 # create running container from image
docker run -it ubuntu /bin/bash                         # create container from ubuntu image and get a bash shell in it

docker ps                                               # get info about running containers
docker ps -a                                            # get info about all containers

docker start <container-id>                             # start a stopped container
docker stop <container-id>                              # stop a running container (SIGTERM -> SIGKILL)
docker attach <container-id>                            # get shell in running container
docker run -dit <image-name> <command>                  # run in detached mode - do not stop until explicitly told
CTRL + P, CTRL + Q                                      # exit container without stopping

docker commit <container-id> <repo>/<image-name>:<tag>  # create image from running container

# --- Port redirection --- #
docker run -dit -p <host-port>:<container-port> <image> <command>
docker run -dit -p 8080:80 ubuntu /bin/bash     # forward container traffic on 80 to host port 8080
```

### Dockerfiles

File with a set of instructions to create a Docker image - a recipe for a Docker image

Steps:
1. Create a dir named after the image that you want to create
2. Inside this dir, create file named `Dockerfile`

```bash
FROM ubuntu                                 # define base image. Downloads from Dockerhub if not local
MAINTAINER Ryan <ryan@ryan.com>             # optional: declaring image author
# Avoid confirmation messages
ARG DEBIAN_FRONTEND=noninteractive          # set env var so package installations do not ask questions - use default answers
# Update the container's packages
RUN apt update; apt dist-upgrade -y         # run these commands when creating the image
# Install apache2 and vim
RUN apt install -y apache2 vim-nox          # install these packages
# Start Apache
ENTRYPOINT apache2ctl -D FOREGROUND         # command to run the app in the container


# --- Create image from Dockerfile --- #
# run in same dir as Dockerfile
docker build -t <repo>/<image-name>:<tag> . # build image from dockerfile - run in dir with dockerfile
docker build -t myrepo/apache-server:1.0 .  # example
```

## LXD (Lex-D)

Descendant of LXC (Lex-C, "Linux Containers") which uses cgroups, but LXD does not replace LCD - only builds on top of it:
- LXD is developed by canonical so available in Ubuntu
  - Also distributed as snap package so it can run on all distros
- LXD expands on LXC by adding snapshots, AFS support, and migration
- Has a filesystem that you can directly access form the host
- LXD is more like a machine container that tries to emulate a VM
- Manage containers with `lxc` command, but commands for LXD layer use `lxd`
- When you add packages to a running container, they are not deleted when the container stops 
  - No layers, so not as fast as Docker
- Default ubuntu container has `ubuntu` user
- always update apt repo when you login
- LXD has its own IP address space for containers
- Like a VM - you can set up bridged network to get IP addr from your DHCP server. Need wired interface connection


```bash
# --- Setup --- #
sudo snap install lxd                       # install lxd snap
usermod -aG lxd <username>                  # add to lxd group
lxd init                                    # initialize new LXD installation
...
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none
Would you like the LXD server to be available over the network? (yes/no) [default=no]: yes
...

# --- Common commands --- #
lxc launch ubuntu:22.04 <container-name>    # downloads and starts container
lxc list                                    # list all containers
lxc start <container-name>                  # start container
lxc stop <container-name>                   # stop container
lxc delete <container-name>                 # delete container
lxc image list                              # list downloaded images
lxc image delete <image-name>               # delete image

# --- Managing containers --- #
lxc exec <container-name> bash                        # execute bash command in container
lxc exect <container-name> -- su --login <username>   # login as specific user
lxc config set my container boot.autostart 1          # config container to start on boot
curl <container-ip-addr>                              # verify container is online
```