---
title: "xVirtualization"
# linkTitle: ""
weight: 140
# description:
---

Ubuntu has a virtualization suite that lets you set up a server that is a centrally available server you can run VMs on:
- Kernel-based VM (KVM) and Quick Emulator (QEMU)
  - KVM is built into the kernal that handles low-level instructins in the kernel to separate host from guest
  - KVM can run at near native speeds
  - QEMU emulates hardware components found in physical servers
- Lets machine run VMs without a 3rd party product like VirtualBox (a centrally available VM server)
  - Can only run VirtualBox or KVM/QEMU, not both at the same time. CPU can't handle both
- Virtual Machine Manager is a GUI that helps you perform VM admin tasks - `virt-manager`
  - Can manage local and remote KVM servers
- KVM machines are assigned IP addr on `192.168.122.0/24` network
  - Get IP from internal DHCP server, between `192.168.122.2` - `192.168.122.254`
  - Can SSH into computer from workstation, set up bridging to get vm to connect directly to your network


## KVM/QEMU Installation 

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo          # check whether cpu supports virtualization extensions
24                                          # more than one means it supports

# --- Installation --- #
# Configure the server, setup workstation to connect to server, workstation to manage virtualization: 
apt install bridge-utils libvirt-clients          # 1. Install libvirt packages to make kvm/qemu work
                         libvirt-daemon-system
                         qemu-system-x86
systemctl status libvirtd                         # 2. check service status
systemctl stop libvirtd                           # 3. stop service for add'l configuration
cat /etc/group | grep kvm                         # 4. verify kvm and libvirtd groups were added
cat /etc/group | grep libvirt
usermod -aG kvm <username>                        # 5. add user to groups
usermod -aG libvirt <username>
# 6. might have to log out and log in to start group memberships
chown :kvm /var/lib/libvirt/images                # 7. give group access to access data in ../images
chmod g+rw /var/lib/libvirt/images                # 8. give rw perms to kvm group
systemctl start libvirtd                          # 9. restart service
apt install ssh-askpass virt-manager              # 10. install GUI for admin tasks
```

### Virtual Machine Manager setup

1. Open Virtual Machine Manager:
   1. Either search in **Applications**, or enter `virt-manager` on command line 
2. Select **File** > **Add Connection** to create a new connection to the VM server.
3. Complete the info in the window. For remote hosts, connect with SSH and add the host info. For a local KVM server, select **Autoconnect**.


### Create a storage pool

Create a new storage pool to store our ISO files:

1. In Virtual Machine Manager, right-click the connection and select **Details**.
2. Select the **Storage** tab.
3. Select the green plus symbol at the bottom-left of the screen.
4. Enter the following info:
   - **Name**: ISO 
   - **Target Path**: `/var/lib/libvirt/images/ISO`
5. Select **Finish**
6. Update permissions to the target path directory:
   ```bash 
   chown root:kvm /var/lib/libvirt/images/ISO
   chmod g+rw /var/lib/libvirt/images/ISO
   ```
7. Move ISO files to the target path so the VM server can create VMs with it.

### Create a VM 

1. In Virtual Machine Manager, right-click the server and select **New**.
2. Leave **Local install media** selected and select **Forward**.
3. Select **Browse** to select the /var/lib/libvirt/images/ISO storage pool. Highlight the ISO and select **Choose Volume**. Select **Forward**.
4. Allocate RAM and CPU:
   - 2048MB should be enough RAM
   - 2 CPUs is plenty
5. Allocate disk space. 20GB is enough for most test machines.
6. Name the VM and select **Finish**.

### Bridging the network

Lets you receive an IP from the DHCP server on your network instead of internal KVM DHCP:
- Good if you're setting up a VM server on your network
- Every VM can be connected to and treated as a regular machine on the network, with the main DHCP server as the source of truth
- Bridging generally works on wired network cards, and does not work on wireless network cards 
  - backup your netplan file before making changes!


### Cloning 

Create a VM, configure it, then convert it into a template for all VMs of the same operating system:
- KVM doesn't have a templating feature, but you can still configure a VM, name it *-template, then clone it

1. Right-click a stopped VM and select **Clone**.
2. Make sure the 'clone this disk' option is selected.
3. Select **Clone**.

### Managing VMs from commmand line 

Use `virsh` suite of commands to manage VMs if GUI isn't available:

```bash
virsh list                    # list running VMs
virsh list --all              # list all VMs
virsh start <vm-name>         # start VM
virsh shutdown <vm-name>      # shutdown VM
virsh suspend <vm-name>       # pause VM
virsh resume <vm-name>        # unpause VM
virsh destroy <vm-name>       # stop VM immediately
virsh undefine <vm-name>      # deletes a VM, but not associated files
                              # must delete disk files from /var/lib/libvirt/images
virsh net-dhcp-leases default

```
### Static IP addresses

Add the static IP config to the default network xml file so the DHCP server can assign the static ip:
- [Official docs](https://wiki.libvirt.org/Networking.html#guest-configuration-nat)

```bash
virsh net-dhcp-leases default                         # view all dhcp leases on default network

# add static IP to xml config file
virsh net-update default add ip-dhcp-host "<host mac='<mac-addr>' name='<hostname>' ip='<ip-addr>' />" --live --config
# delete static IP
virsh net-update default delete ip-dhcp-host "<host mac='<mac-addr>' name='<hostname>' ip='<ip-addr>' />" --live --config

virsh net-list
virsh net-dumpxml default
virsh dumpxml <vm-name> | grep -i '<mac'      # get mac addr
virsh net-edit <network-name>                 # edit network config
```