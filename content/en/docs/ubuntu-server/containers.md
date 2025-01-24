---
title: "Containers"
# linkTitle: ""
weight: 150
# description:
---

- Containers are isolated filesystems
  - VMs require that we dedicate resources. We provision CPUs, RAM, and disk space - it might not use all allocated resources
  - Containers share CPU, kernel with the host
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


```bash
docker search <keyword>                 # search for pre-existing image
docker pull <image-name>                # pull docker image to local machine
docker images                           # list docker images on your server
docker rmi <image-id>                   # remove image from your server
docker rm <container-id>                # remove persistent container
docker run <image-name>                 # create running container from image
docker run -it ubuntu /bin/bash         # create container from ubuntu image and get a bash shell in it
docker ps                               # get list of running containers
docker ps -a                            # get list of all containers

```

## LXD (Lex-D)

Descendant of LXC (Lex-C, "Linux Containers") which uses cgroups, but LXD does not replace LCD - only builds on top of it:
- LXD is developed by canonical so available in Ubuntu
  - Also distributed as snap package so it can run on all distros
- LXD expands on LXC by adding snapshots, AFS support, and migration
- Has a filesystem that you can directly access form the host
- LXD is more like a machine container that tries to emulate a VM