+++
title = 'Virtualization'
date = '2025-09-07T19:07:59-04:00'
weight = 10
draft = false
+++


Ubuntu includes a virtualization suite built on *KVM* (Kernel-based VM) and *QEMU* (Quick Emulator). KVM is built into the Linux kernel and separates the host from guest operating systems, enabling near-native performance. QEMU emulates the hardware components found in physical servers.

KVM/QEMU lets you run virtual machines without a third-party product like VirtualBox. You cannot run VirtualBox and KVM/QEMU simultaneously — the CPU cannot support both hypervisors at the same time.

*Virtual Machine Manager* (`virt-manager`) is a GUI for VM administration. It can manage both local and remote KVM servers.

KVM assigns VMs addresses on the `192.168.122.0/24` network from an internal DHCP server (`192.168.122.2`–`192.168.122.254`). You can SSH into VMs from your workstation, or configure bridging to connect VMs directly to your local network.


## KVM/QEMU installation

Before installing, verify that your CPU supports virtualization extensions. A result greater than zero confirms support:

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

To install and configure KVM/QEMU:

1. Install the required packages:
   ```bash
   apt install bridge-utils libvirt-clients \
               libvirt-daemon-system qemu-system-x86
   ```
2. Check the service status:
   ```bash
   systemctl status libvirtd
   ```
3. Stop the service for additional configuration:
   ```bash
   systemctl stop libvirtd
   ```
4. Verify that the `kvm` and `libvirt` groups were created:
   ```bash
   grep kvm /etc/group
   grep libvirt /etc/group
   ```
5. Add your user to both groups. Log out and back in for group membership to take effect:
   ```bash
   usermod -aG kvm <username>
   usermod -aG libvirt <username>
   ```
6. Grant the `kvm` group access to the images directory:
   ```bash
   chown :kvm /var/lib/libvirt/images
   chmod g+rw /var/lib/libvirt/images
   ```
7. Start the service:
   ```bash
   systemctl start libvirtd
   ```
8. Install the Virtual Machine Manager GUI:
   ```bash
   apt install ssh-askpass virt-manager
   ```

### Virtual Machine Manager setup

Complete this setup on your workstation after installing KVM/QEMU on the server. Virtual Machine Manager gives you a GUI to create, start, stop, and manage VMs on local or remote KVM hosts:

1. Open Virtual Machine Manager by searching in **Applications** or running `virt-manager`.
2. Select **File** > **Add Connection**.
3. For a remote host, select SSH and enter the host details. For a local KVM server, select **Autoconnect**.


### Create a storage pool

A storage pool gives Virtual Machine Manager a managed location to find ISO images when creating VMs. Create one the first time you set up a KVM server so you have a consistent place to store installation media:

1. In Virtual Machine Manager, right-click the connection and select **Details**.
2. Select the **Storage** tab.
3. Select the green plus icon at the bottom left.
4. Enter the following:
   - **Name**: ISO
   - **Target Path**: `/var/lib/libvirt/images/ISO`
5. Select **Finish**.
6. Set permissions on the new directory:
   ```bash
   chown root:kvm /var/lib/libvirt/images/ISO
   chmod g+rw /var/lib/libvirt/images/ISO
   ```
7. Move your ISO files into the target path so the VM server can use them to create VMs.

### Create a VM

Use Virtual Machine Manager to provision a new VM from an ISO image stored in your storage pool:

1. In Virtual Machine Manager, right-click the server and select **New**.
2. Leave **Local install media** selected and select **Forward**.
3. Select **Browse**, navigate to the `/var/lib/libvirt/images/ISO` storage pool, select your ISO, and select **Choose Volume**. Select **Forward**.
4. Allocate at least 2048 MB of RAM and 2 CPUs.
5. Allocate disk space. 20 GB is sufficient for most test machines.
6. Enter a name for the VM and select **Finish**.

### Bridging the network

By default, KVM assigns VMs addresses from its internal DHCP server. Bridging connects VMs directly to your local network so they receive addresses from your main DHCP server and are reachable like any other machine. This is useful when you want other devices on the network to communicate with your VMs.

Bridging requires a wired network interface. It does not work on wireless cards. Back up your Netplan configuration before making changes.


### Cloning

Cloning lets you replicate a configured VM without repeating the full installation and setup process. KVM does not have a built-in templating feature, but you can configure a base VM, name it with a `-template` suffix, and clone it whenever you need a new instance:

1. Right-click a stopped VM and select **Clone**.
2. Confirm that **Clone this disk** is selected.
3. Select **Clone**.

### Managing VMs from the command line

Use `virsh` to manage VMs when Virtual Machine Manager is not available, such as on a headless server or over SSH:

```bash
virsh list                    # list running VMs
virsh list --all              # list all VMs
virsh start <vm-name>         # start a VM
virsh shutdown <vm-name>      # shut down a VM gracefully
virsh suspend <vm-name>       # pause a VM
virsh resume <vm-name>        # resume a paused VM
virsh destroy <vm-name>       # force-stop a VM immediately
virsh undefine <vm-name>      # remove a VM definition (disk files remain in /var/lib/libvirt/images)
```
### Static IP addresses

Assign a static IP to a VM by adding a DHCP host entry to the default network configuration. This ensures the VM always receives the same address from KVM's internal DHCP server. See the [official docs](https://wiki.libvirt.org/Networking.html#guest-configuration-nat) for reference.

Look up the VM's MAC address and current leases, then update the network configuration:

```bash
virsh net-dhcp-leases default                         # view all DHCP leases on the default network
virsh dumpxml <vm-name> | grep -i '<mac'              # get the VM's MAC address
virsh net-list                                        # list available networks
virsh net-dumpxml default                             # view the default network XML config
virsh net-edit <network-name>                         # edit network config directly
```

Add or remove a static IP entry:

```bash
# add a static IP
virsh net-update default add ip-dhcp-host \
    "<host mac='<mac-addr>' name='<hostname>' ip='<ip-addr>' />" --live --config

# remove a static IP
virsh net-update default delete ip-dhcp-host \
    "<host mac='<mac-addr>' name='<hostname>' ip='<ip-addr>' />" --live --config
```