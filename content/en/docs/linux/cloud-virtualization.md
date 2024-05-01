---
title: "Cloud and virtualization"
weight: 230
---

## Cloud computing

It's called _cloud_ because the original ARPAnet network docs used a cloud symbol to represent a large network of interconnected servers.

_Cloud computing_ is _distributed computing_, where resources are shared among two or more servers to accomplish a single task, such as run an application. Resources are commonly delivered across the internet. There are three kinds:
- Public: A third party provides all computing resources outside the organization.
- Private: Each org builds its own cloud computing resources to provide resources internally.
- Hybrid: Mix of public and private. Public cloud usually supplements resources.

### Cloud services

Cloud computing vendors provide the following services:

IaaS
: Infrastructure-as-a-service model, where the vendor provides low-level resources to host apps. Low-level means CPU time, memory space, storage, etc. The customer provides the OS. The resources might be spread across geographies to increase availability.

PaaS
: Platform-as-a-service model, where the vendor provides the physical server env as well as the OS to the customer and manages updates and security patches.

SaaS
: Software-as-a-service model, where the vendor provides a complete application environment, such as a web server. The vendor provides the physical server, the OS, and the application software.

## Virtualization

### Hypervisors

Runs multiple virtual smaller server environments on a single physical server. Each virtual server--_virtual machine_-- operates as a stand-alone server running on the physical server hardware.

The _hypervisor_, also called the _virtual machine monitor_ (VMM), provides a virtual environment of resources (CPU, storage, memory) to each VM running on the server. The servers don't know about the hypervisor or the other servers on the same physical machine.

### Type I hypervisor

Commonly called _bare-metal hypervisors_. The hypervisor runs directly on the server hardware and allocates resources to each VM, as needed. There are two popular type I hypervisors in Linux:
- **KVM**: Linux Kernel-based Virtual Machine that uses a standard linux kernel with a hypervisor module. Supports Intel and AMD.
- **XEN**: The XEN project is an open source standard for hardware virtualization. Supports Intel, AMD, and ARM CPUs.

### Type II hypervisor

Commonly called _hosted hypervisors_ because they run guest virtual machines on top of an existing OS installation. Popular type II hypervisors include VMware Workstation, QEMU, and Oracle VirtualBox.

### Hypervisor templates

You have to configure the VMs that you create on the hypervisor, which includes provisioning underlying hardware resources. You can save these settings to template files so that you can easily duplicate a VM environment on either the same or separate hypervisor.
- Open Virtualization Format (OVF) is the open source standard for for VM configs. Uses XML files that are difficult to distribute.
- Open Virtualization Applicance (OVA) template format bundles all OVF files into a single tar archive for easy distribution.


## Containers

Applications require lots of files:
- runtime files, usually stored in a single dir
- library files for db, desktop, or OS interfaces that are usually scattered throughout the virtual filesystem

Containers gather all these files in a self-contained application environment. This means that you can deploy the app with no issues in any environment that supports containers.
- Don't have the entire OS, so they are lightweight compared to VMs
- Use `chroot jail` and incorporate security features, kernel namespaces, control groups (cgroups), and other kernel capabilities.

### Container software

Two main container packages used in Linux:
- **LXC**: Developed as an open source standard for creating containers.
  - Include their own bare-bones OS that interfaces with the host system hardware directly, so it doesn't require that a host OS manage resources
  - Sometimes referred to as VMs bc they include a mini-OS
- **Docker**: Open source and extremely lightweight.
  - Uses a separate daemon that runs on Linux and manages the Docker images
  - Daemon listens to requests from containers and the Docker CLI to control the containers

### Container templates

Create templates to easily duplicate a container:
- LXC uses _LXD_
- Docker uses the Docker _container image_ file, which is a read-only container image that can store and distribute app containers.

### Docker

Docker shares the host OS kernel with the containers, so you can only run Linux containers on a Linux host:J

```bash
# check docker is running
docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# pull image 
docker pull docker.io/library/httpd:latest
latest: Pulling from library/httpd
...
Digest: sha256:36c8c79f900108f0f09fd4148ad35ade57cba0dc19d13f3d15be24ce94e6a639
Status: Downloaded newer image for httpd:latest
docker.io/library/httpd:latest

# create container named myApache
$ docker run -d -t -p 8088:80 --name myApache httpd
085ec6cee0055368966d9e2ffcfe658d731a96479a7c3a85019deef4ef622898

# verify image is running
$ docker ps
CONTAINER ID   IMAGE     COMMAND              CREATED          STATUS          PORTS                                   NAMES
085ec6cee005   httpd     "httpd-foreground"   17 seconds ago   Up 16 seconds   0.0.0.0:8088->80/tcp, :::8088->80/tcp   myApache

# get shell in container
$ docker exec -i -t myApache bash
root@085ec6cee005:/usr/local/apache2# ls
bin  build  cgi-bin  conf  error  htdocs  icons  include  logs	modules
root@085ec6cee005:/usr/local/apache2# ls /usr/local/apache2/htdocs/
index.html
root@085ec6cee005:/usr/local/apache2# cat /usr/local/apache2/htdocs/index.html 
<html><body><h1>It works!</h1></body></html>
root@085ec6cee005:/usr/local/apache2# exit
exit

# stop container
$ docker stop myApache 
myApache

# verify container is stopped
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# rm container
$ docker rm myApache 
myApache

# rm image
$ docker rmi httpd
Untagged: httpd:latest
Untagged: httpd@sha256:36c8c79f900108f0f09fd4148ad35ade57cba0dc19d13f3d15be24ce94e6a639
...
```

## VM tools

VM utilities let you create, destroy, boot, shut down, and configure your guest VMs.

### libvirt

A primary goal of the `libvirt` roject is t provide a single way to manage VMs. Supports the following hypervisors:
- KVM
- QEMU
- Xen
- VMware
- ESXi

```bash
virsh
Welcome to virsh, the virtualization interactive terminal.

Type:  'help' for help with commands
       'quit' to quit

virsh # help
Grouped commands:

 Domain Management (help keyword 'domain'):
    attach-device                  attach device from an XML file
    attach-disk                    attach disk device
    attach-interface               attach network interface
    autostart                      autostart a domain
    ...
```

### virt-manager

Virtual Machine Manager (VMM)--_not_ the hypervisor--is a lightweight desktop app for creating and managing virtual machines.

## Storage issues

Provisioning
: When you create a VM, you have to select the amount of disk storage. Virtual disks are either thin or thick provisioned:
  - **Thick provisioning**: Static, and the physical files created on the physical disk are preallocated. If you select 500GB as the virtual disk size, 500GB of physical disk space is reserved.
  - **Thin provisioning**: Dynamic, so the VM can consume for the virtual drive the disk space that it uses. If you select 500GB as the virtual disk size but then only use 100GB, then 100GB of physical disk space is used. As you use more data, you take up more physical disk space until you reach 500GB.
    
    When you thin provision, you might _overprovision_, which means assign more virtual disk space than you have available physical disk.

Persistent volumes
: Similar to a physical disk--data is kept on disk until deleted or overwritten and persists even after VM is destroyed.

Blobs
: Azure cloud platform term for unstructured data that can be manipulated with .NET code. Blob data can be of the following types:
  - Block blob: Blocks of binary and text data, where each block is limited to 4.7TB.
  - Append blob: Blocks of binary and text data, and storage is enhanced for appending operations, which makes these good for logging.
  - Page blobs: Random access files up to 8TB. Used as virtual disks for Azure VMs.

## Network configurations

VMs can have any number of virtualized NICs, virtualized switches, firewalls, load balancers, etc. Important concepts are VLANs and overlay networks:

VLAN
: Think of LANs (local area networks), which are systems and various devices typically located in a small area, like a building. They share common communications line or wireless link and are often broken up into different netowrk segments. Network traffic is fast.
  
  VLANs are systems and devices that are located across various LAN subnets. VLANs are not based on physical location, they use logical and virtualized connections and use layer 2 to broadcast messages (LANs use routers on layer 3 for broadcasting).

Overlay network
: Cheap and scalable virtualization method that uses encapsulation and channel tunneling. A network connection (wired or wireless) is virtually split into different channels, and each channel is assigned to a service or device. Packets in the channel are encapsulated in a packet, which is removed when the packet reaches its destination. 

  Applications manage the network ifrastructure. There are virtual switches, tunneling protocols, and softare-defined-networking (SDN), which is a method for controlling and managing network communications via software. There is an SDN controller program and two APIs called northbound and southbound that control traffic.

### Virtualized NICs

Sometimes the VM is connected to the host NIC, and sometimes it is connected to a virtualized switch. You have lots of options:

Host-Only
: Sometimes called _local adapter_, connects to a virtual netowrk contained withi the VMs host system. There is no connection to any external network that the host is attached to.

  This is fast, bc traffic travels through RAM, not network components. Example is a proxy server.

Bridged
: The VM is like a node on the LAN or VLAN that the host system is attached. The VM gets its own IP address and is available on the network.

NAT
: Network Address Translation adapter, which uses a network device like a router to 'hide' a LAN computer's IP address when it sends traffic to antoher network. ALl LAN addresses are translated into a single IP address through the NAT device. When data is received, the NAT router sends it to the correct LAN host.

  In VMs, NAT tables are maintained by the hypevisor, not a router or network device. The host IP is sent out to external networks, and the VMs have an IP address withi the host's virtual network.

Dual-Homed
: A computer with one or more active network adapters (multiple NICs) for redundancy.

  A virtual prozy server is dual-homed: one internal network NIC (host-only) and a bridged adapter to send data over the external network.