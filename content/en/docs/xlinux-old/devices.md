---
title: "Linux devices"
linkTitle: "Devices"
weight: 210
---

The kernel must recognize and know how to communicate with each device on your system. The kernel communicates with devices using modules.

## Device interfaces

Each device uses a standard protocol to communicate with the system hardware. The kernel module must know how to send data to and receive data from the device with this protocol. There are three popular standards to connect devices:
- PCI boards
- USB
- GPIO

### PCI boards

Peripheral Component Interconnect (PCI) was developed to connect hardware boards to PC motherboards. Now, it is commonly used to provide an interface for external hardware cards.

Some devices that use PCI boards:
- **Internal hard drives**: Hard drives that use SATA and SCI connectors. The kernel automatically recognizes both of these hard drive types.
- **External hard drives**: Network hard drives that use the Fibre Channel standard use PCI boards that support the Host Bus Adapter (HBA) standard.
- **Network Interface Cards**: Hard-wired cards let you connect to the network with the RJ-45 cable standard.
- **Wireless Cards**: PCI boards that support IEEE 802.11. Popular in workstations more than servers.
- **Bluetooth Devices**: Short-distance wireless communication with other bluetooth devices in peer-to-peer network.
- **Video Accelerators**: Apps that need advanced graphics use video accelerator cards, that offload video processing from the CPU. 
- **Audo Cards**: Apps that need high-quality sound use special audio cards, that offload audio processing from the CPU.

### USB interface

Universal Serial Bus (USB) uses serial connections, so it requires fewer connectors to the motherboard which allows for a smaller interface. USB 4.0 has data transfer speeds of 40 Gbps.

For linux to communicate with USB:
1. Kernel must have module to recognize USB controller on the machine. The controller helps the kernel and the machine's USB bus communicate
2. Kernel needs module for individual device type you plug into the USB bus

### GPIO

General-purpose input/output (GPIO) interface is popular with small utility Linux systems that are designed to control external devices for automation projects, like Raspberry Pi. Used to control objects and environments:
- control room temp
- sense when doors or windows open/close
- sense motion
- control robot operations

## /dev directory

Stores device files. A device file is a special file that linux uses to interface with hardware devices--they allow the system to transfer data to and from the device.
- Writes to device file to send data to device
- Reads device file to retrieve data from device

### Device files

Two types of device files:

- **Character**: Transfers data one char at a time. Often used for serial devices such as terminals or USB devices
- **Block**: Transfers large blocks of data. Used for high-speed data transfer devices like hard drives or network cards.


Block starts with `b`, character starts with `c`:

```bash
ls -al /dev/sd* /dev/tty*
ls: cannot access '/dev/sd*': No such file or directory
crw-rw-rw- 1 root        tty     5,  0 Apr 25 08:29  /dev/tty
crw--w---- 1 root        tty     4,  0 Apr 18 21:53  /dev/tty0
crw--w---- 1 root        tty     4,  1 Apr 18 21:53  /dev/tty1
crw--w---- 1 root        tty     4, 10 Apr 18 21:53  /dev/tty10
...
```

### Special device files

| File | Description |
|---|---|
| `/dev/null` | Data sink. Send data that you want to discard here. |
| `/dev/random` | Access to kernel's random number generator. Blocks requests until enough random data has been generated to calculate a true random number. |
| `/dev/urandom` | Access to kernel's random number generator. Does not block, returns random number. Less accurate than `/dev/random` |
| `/dev/zero` | When data is read from this device, it returns a NULL character (`0x00`). Used to create null files, or erase data on a disk partition. |


```bash
ls -l /dev/null 
crw-rw-rw- 1 root root 1, 3 Apr 18 21:53 /dev/null

ls -l /dev/*rand*
crw-rw-rw- 1 root root 1, 8 Apr 18 21:53 /dev/random
crw-rw-rw- 1 root root 1, 9 Apr 18 21:53 /dev/urandom
 
ls -l /dev/zero 
crw-rw-rw- 1 root root 1, 5 Apr 18 21:53 /dev/zero
```

### device mapper

The kernel maps physical block devices to virtual block devices. The virtual block devices allow the system to intercept the data written to or read from the the physical device and perform some operation on them. Used by:
- LVM to create logical drives
- LUKS to encrypt data

## /proc directory

A virtual directory that the kernel dynamically populates to provide access to information about the system hardware settings and status. Different files in `/proc` track different system features.


### Interrupt requests

Interrupt requests (IRQs) let hardware devices indicate when they have data that they need to send to the CPU. The Plug and Play (PnP) system in linux assigns each device a unique IRQ address, which is stored in the `/proc/interrupts` file.

The number in the first column is the IRQ assigned to the device. Some are default (`0` for the system timer), and some are assigned when the system detects a device during boot:

```bash
cat /proc/interrupts 
            CPU0       CPU1       CPU2       CPU3       CPU4       CPU5       CPU6       CPU7       CPU8       CPU9       CPU10      CPU11      
   1:     242119          0          0          0          0          0          0          0          0          0          0      15983  IR-IO-APIC    1-edge      i8042
   8:          0          0          0          0          0          0          0          0          0          0          0          0  IR-IO-APIC    8-edge      rtc0
   9:     833609      71145          0          0          0          0          0          0          0          0          0          0  IR-IO-APIC    9-fasteoi   acpi
  12:         60          0          0          0          0          0          0          0          0          0         53          0  IR-IO-APIC   12-edge      i8042
  14:          0          0          0          0          0          0          0          0          0          0          0          0  IR-IO-APIC   14-fasteoi   INT3450:00
  16:          0          0          0          3          0          0          0          0          0          0          0          0  IR-IO-APIC   16-fasteoi   idma64.0, i801_smbus, i2c_designware.0
  17:    4099961          0   12223733   20469941          0          0     464405    5467918   12848240          0          0       1074  IR-IO-APIC   17-fasteoi   idma64.1, i2c_designware.1
...
```

### I/O ports

System I/O ports are locations in memory where the CPU can send data to and receive data from the hardware device. The system assigns each device an I/O port, which is tracked in the `/proc/ioports` file:

```bash
cat /proc/ioports 
0000-0000 : PCI Bus 0000:00
  0000-0000 : dma1
  0000-0000 : pic1
  0000-0000 : timer0
  0000-0000 : timer1
  0000-0000 : keyboard
  0000-0000 : keyboard
...
```

If there is a port conflict, you can override the assigned port with the `setpci` command.

### Direct memory access

Sending data through an I/O port can be slow, so some devices use direct memory acces (DMA) to send data from a hardware device directly to memory on the system, without having to wait for the CPU. The CPU can read the data from memory when it is ready.

Each hardware device that uses DMA must be assigned a channel number, which is tracked in `/proc/dma`:

```bash
cat /proc/dma 
 4: cascade
```

## /sys directory

Virtual directory that stores information about hardware devices that any user on the system can access. Each subdirectory is based on a device or system function:

```bash
ls -al /sys/
total 4
dr-xr-xr-x  13 root root    0 Apr 18 21:52 .
drwxr-xr-x  21 root root 4096 Mar 23  2023 ..
drwxr-xr-x   2 root root    0 Apr 18 21:52 block
drwxr-xr-x  52 root root    0 Apr 18 21:52 bus
drwxr-xr-x  85 root root    0 Apr 18 21:52 class
drwxr-xr-x   4 root root    0 Apr 18 21:52 dev
drwxr-xr-x  29 root root    0 Apr 18 21:52 devices
drwxr-xr-x   6 root root    0 Apr 18 21:52 firmware
drwxr-xr-x   8 root root    0 Apr 18 21:52 fs
drwxr-xr-x   2 root root    0 Apr 18 21:53 hypervisor
drwxr-xr-x  16 root root    0 Apr 18 21:52 kernel
drwxr-xr-x 341 root root    0 Apr 18 21:52 module
drwxr-xr-x   3 root root    0 Apr 18 21:53 power
```

## Working with devices

### lsdev

Displays informatin about the hardware devices installed on the system. Combines info from the following files:
- `/proc/interrupts`
- `/proc/ioports`
- `/proc/dma`

```bash
lsdev
Device            DMA   IRQ  I/O Ports
------------------------------------------------
0000:00:02.0                   0000-0000
0000:00:17.0                   0000-0000   0000-0000   0000-0000
0000:00:1f.4                   0000-0000
0000:01:00.0                     0000-0000
acpi                      9 
ACPI                           0000-0000   0000-0000   0000-0000   0000-0000   0000-0000
ahci                             0000-0000     0000-0000     0000-0000
ahci[0000:00:17.0]        131 
cascade             4       
dma                            0000-0000
...
```

### lsblk

Displays information about the block devices installed on the system. Has command-line options:

```bash
# view all block devices
lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0         7:0    0     4K  1 loop /snap/bare/5
loop1         7:1    0  66.1M  1 loop /snap/cups/1044
loop2         7:2    0 160.6M  1 loop /snap/chromium/2805
loop3         7:3    0 105.4M  1 loop /snap/core/16574
...

# view SCSI block devices only
lsblk -S
NAME HCTL       TYPE VENDOR   MODEL      REV SERIAL                                                  TRAN
sda  3:0:0:0    disk  USB     SanDisk 3 1.00 0401095f84926a9e2f38e3ecb97f3b66fe6dc10ae15b2eb3fee86e9 usb
```

### dmesg

View kernel ring buffer records, which are kernel-level events. Because it is a ring, this file overwrites older messages with new ones. Great for troubleshooting kernel modules.

```bash
# verify USB connection
sudo dmesg | tail -20
[91877.590512] [UFW BLOCK] IN=wlp59s0 OUT= MAC=01:00:5e:00:00:01:90:58:51:d4:02:6a:08:00 SRC=10.0.0.1 DST=224.0.0.1 LEN=28 TOS=0x00 PREC=0xC0 TTL=1 ID=12912 PROTO=2 
[91883.121258] [UFW BLOCK] IN=wlp59s0 OUT= MAC=01:00:5e:00:00:fb:84:c5:a6:17:8b:58:08:00 SRC=10.0.0.145 DST=224.0.0.251 LEN=32 TOS=0x00 PREC=0x00 TTL=1 ID=17897 PROTO=2 
[91906.743226] [UFW BLOCK] IN=wlp59s0 OUT= MAC=01:00:5e:00:00:fb:90:58:51:d4:02:6a:08:00 SRC=10.0.0.1 DST=224.0.0.251 LEN=28 TOS=0x00 PREC=0xC0 TTL=1 ID=18127 PROTO=2 
[91906.743262] [UFW BLOCK] IN=wlp59s0 OUT= MAC=01:00:5e:00:00:fb:90:58:51:d4:02:6a:08:00 SRC=10.0.0.1 DST=224.0.0.251 LEN=28 TOS=0x00 PREC=0xC0 TTL=1 ID=18128 PROTO=2 
[91913.497411] usb 2-1: new SuperSpeed USB device number 5 using xhci_hcd
[91913.518503] usb 2-1: New USB device found, idVendor=0781, idProduct=5581, bcdDevice= 1.00
[91913.518515] usb 2-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[91913.518521] usb 2-1: Product:  SanDisk 3.2Gen1
[91913.518526] usb 2-1: Manufacturer:  USB
[91913.518530] usb 2-1: SerialNumber: 0401095f84926a9e2f38e3ecb97f3b66fe6dc10ae15b2eb3fee86e909773f274bd93000000000000000000002d440ab8ff025e18815581076da82bb8
[91913.521064] usb-storage 2-1:1.0: USB Mass Storage device detected
[91913.521752] scsi host3: usb-storage 2-1:1.0
[91914.547750] scsi 3:0:0:0: Direct-Access      USB      SanDisk 3.2Gen1 1.00 PQ: 0 ANSI: 6
[91914.548442] sd 3:0:0:0: Attached scsi generic sg0 type 0
[91914.548960] sd 3:0:0:0: [sda] 120176640 512-byte logical blocks: (61.5 GB/57.3 GiB)
[91914.550065] sd 3:0:0:0: [sda] Write Protect is off
[91914.550074] sd 3:0:0:0: [sda] Mode Sense: 43 00 00 00
[91914.550545] sd 3:0:0:0: [sda] Write cache: disabled, read cache: enabled, doesn\'t support DPO or FUA
[91914.587927]  sda:
[91914.590460] sd 3:0:0:0: [sda] Attached SCSI removable disk
```

### lspci

View the currently installed and recognized PCI and PCIe devices.Useful to troubleshoot PCI card issues, like when it is not recognized by the system.

[lspci useful options](https://phoenixnap.com/kb/lspci-command#ftoc-heading-3)

```bash
lspci
00:00.0 Host bridge: Intel Corporation 8th Gen Core Processor Host Bridge/DRAM Registers (rev 07)
00:01.0 PCI bridge: Intel Corporation 6th-10th Gen Core Processor PCIe Controller (x16) (rev 07)
00:02.0 VGA compatible controller: Intel Corporation CoffeeLake-H GT2 [UHD Graphics 630]
00:04.0 Signal processing controller: Intel Corporation Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor Thermal Subsystem (rev 07)
00:08.0 System peripheral: Intel Corporation Xeon E3-1200 v5/v6 / E3-1500 v5 / 6th/7th/8th Gen Core Processor Gaussian Mixture Model
00:12.0 Signal processing controller: Intel Corporation Cannon Lake PCH Thermal Controller (rev 10)
00:14.0 USB controller: Intel Corporation Cannon Lake PCH USB 3.1 xHCI Host Controller (rev 10)
00:14.2 RAM memory: Intel Corporation Cannon Lake PCH Shared SRAM (rev 10)
00:15.0 Serial bus controller: Intel Corporation Cannon Lake PCH Serial IO I2C Controller #0 (rev 10)
00:15.1 Serial bus controller: Intel Corporation Cannon Lake PCH Serial IO I2C Controller #1 (rev 10)
...
```

### lsusb

View basic info about USB devices connected to the system:

```bash
lsb [OPTIONS]
-d # specify vendor ID
-D # specify device file
-s # specify bus
-t # display in tree format, showing related devices
-v # verbose
-V # version

# standard output
lsusb
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 005: ID 0781:5581 SanDisk Corp. Ultra
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 003: ID 27c6:5395 Shenzhen Goodix Technology Co.,Ltd. Fingerprint Reader
Bus 001 Device 038: ID 8087:0029 Intel Corp. AX200 Bluetooth
Bus 001 Device 004: ID 0c45:6723 Microdia Integrated_Webcam_HD
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

# tree view
lsusb -t
/:  Bus 04.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/2p, 10000M
/:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/2p, 480M
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/10p, 10000M
    |__ Port 1: Dev 5, If 0, Class=Mass Storage, Driver=usb-storage, 5000M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/16p, 480M
    |__ Port 4: Dev 38, If 1, Class=Wireless, Driver=btusb, 12M
    |__ Port 4: Dev 38, If 0, Class=Wireless, Driver=btusb, 12M
    |__ Port 7: Dev 3, If 0, Class=Communications, Driver=, 12M
    |__ Port 7: Dev 3, If 1, Class=CDC Data, Driver=, 12M
    |__ Port 12: Dev 4, If 1, Class=Video, Driver=uvcvideo, 480M
    |__ Port 12: Dev 4, If 0, Class=Video, Driver=uvcvideo, 480M
```

## Monitor support

Linux controls the video environment with two components:
- video card
- monitor

### X Window System 

_X Window System_ is a standard protocol for interacting with displays. Commonly referred to as X or X11 (latest version). Newest software pacakges:
- **X.org**: Uses simple text-based config files in `/etc/X11`
- **Wayland**: Simple and secure, developed by Red Hat. Uses separate config files for each user in the `~/.config/weston.ini` config file in each user home dir.

Both softwares detect the video card at boot time and make changes. They can also detect video equiptment on the fly and make changes as needed.

```bash
# X.org config files
ls /etc/X11/
app-defaults             fonts    xkb          Xreset.d    Xsession.d        XvMCConfig
cursors                  rgb.txt  xorg.conf.d  Xresources  Xsession.options  Xwrapper.config
default-display-manager  xinit    Xreset       Xsession    xsm
```

## Printers

> Find linux-compatible printers:
> - [Open Printing](https://openprinting.org/printers)
> - [The Printing HOWTO](https://tldp.org/HOWTO/Printing-HOWTO/printers.html)


Linux manages printer drivers with the Common Unix Printing System (CUPS), available on the workstation at port 361 (`localhost:631`). You can configure printers using different protocols.

### Print queue commands

```bash
# start, stop or pause print queue
lpc -P <printer>

# display print queue status and pending jobs
lpq -P <printer>

# submit new print job
lpr -P <printer> <file>

# rm specific print job from queue
lprm -P <printer>
```

## Hot-pluggable devices

Hardware is generally categorized in one of two ways:
- **Cold-pluggable devices**: Must be connected when the system is off. Generally found in the computer case, such as:
  - memory
  - PCI cards
  - hard drives
- **Hot-pluggable devices**: External components, can add or remove at any time, such as:
  - network connections
  - monitors
  - USB devices

## Detecting dynamic devices

_`udev device manager`_ listens to kernel notifications about hardware devices that are plugged in or removed. Its started at boot time by the `init` process and always runs in the background.

`udev device manager` listens to notifications and compares the messages against rules set in `/etc/udev/rules.d`:

```bash
# virtualbox udev rules file
cat /etc/udev/rules.d/60-vboxadd.rules 
KERNEL=="vboxguest", NAME="vboxguest", OWNER="vboxadd", MODE="0660"
KERNEL=="vboxuser", NAME="vboxuser", OWNER="vboxadd", MODE="0666"
```
### udevadm

Interacts with the `udev device manager`:

```bash
udevadm command [OPTIONS]

# view info about USD drive
udevadm info /dev/sda
P: /devices/pci0000:00/0000:00:0d.0/ata3/host2/target2:0:0/2:0:0:0/block/sda
N: sda
L: 0
S: disk/by-path/pci-0000:00:0d.0-ata-1.0
S: disk/by-id/ata-VBOX_HARDDISK_VBc96b84e1-50baf623
...
```

## Troubleshooting

### dmidecode

> DO NOT rely on `dmidecode` alone for hardware information, especially when running a virtual machine.

The Distributed Management Task Force (DMTF) is a nonprofit whose goal is to simplify the management of network-accessible technologies through standards. Created the following standards:
- **Desktop Management Interface (DMI)**: Consists of four components that provide information about the hardware being used on a computer.
- **System Management BIOS (SMBIOS)**: items such as datastructures that read management info produces by a system's BIOS.

You can interact with these standards with the `dmidecode` utility. It pulls info from the `sysfs` filesystem, specifically the `/sys/firmware/dmi/tables/` dir:

```bash
# view sysfs tables
ls /sys/firmware/dmi/tables/
DMI  smbios_entry_point

# view help
dmidecode -h
Usage: dmidecode [OPTIONS]
Options are:
 -d, --dev-mem FILE     Read memory from device FILE (default: /dev/mem)
 -h, --help             Display this help text and exit
 -q, --quiet            Less verbose output
 -s, --string KEYWORD   Only display the value of the given DMI string
 -t, --type TYPE        Only display the entries of given type
 -H, --handle HANDLE    Only display the entry of given handle
 -u, --dump             Do not decode the entries
     --dump-bin FILE    Dump the DMI data to a binary file
     --from-dump FILE   Read the DMI data from a binary file
     --no-sysfs         Do not attempt to read DMI data from sysfs files
     --oem-string N     Only display the value of the given OEM string
 -V, --version          Display the version and exit
```

`-t, --type TYPE` arguments:
- `baseboard`
- `bios`
- `cache`
- `chassis`
- `connector`
- `memory`
- `processor`
- `slot`
- `system`

```bash
# view memory info
sudo dmidecode -t memory
# dmidecode 3.3
Getting SMBIOS data from sysfs.
SMBIOS 3.2.0 present.

Handle 0x0037, DMI type 16, 23 bytes
Physical Memory Array
	Location: System Board Or Motherboard
	Use: System Memory
	Error Correction Type: None
	Maximum Capacity: 32 GB
	Error Information Handle: Not Provided
	Number Of Devices: 2

Handle 0x0044, DMI type 17, 84 bytes
Memory Device
	Array Handle: 0x0037
	Error Information Handle: Not Provided
	Total Width: 64 bits
	Data Width: 64 bits
	Size: 16 GB
...

# view system table info
sudo dmidecode -t system
# dmidecode 3.3
Getting SMBIOS data from sysfs.
SMBIOS 3.2.0 present.

Handle 0x0001, DMI type 1, 27 bytes
System Information
	Manufacturer: Dell Inc.
	Product Name: Precision 5540
	Version: Not Specified
	Serial Number: GHWCP73
...
```

### lshw

Retrieves information about your hardware from the `/proc` directory files. Useful options include `-short`, `-businfo`, and `-class`:

```bash
# table-formatted hardware data
lshw -short
H/W path           Device          Class          Description
=============================================================
                                   system         Precision 5540 (0906)
/0                                 bus            07C17G
/0/0                               memory         64KiB BIOS
/0/37                              memory         32GiB System Memory
/0/37/0                            memory         16GiB SODIMM DDR4 Synchronous 2667 MHz (0.4 ns)
/0/37/1                            memory         16GiB SODIMM DDR4 Synchronous 2667 MHz (0.4 ns)
/0/40                              memory         384KiB L1 cache
/0/41                              memory         1536KiB L2 cache
/0/42                              memory         12MiB L3 cache
/0/43                              processor      Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
/0/100                             bridge         8th Gen Core Processor Host Bridge/DRAM Registers
/0/100/1                           bridge         6th-10th Gen Core Processor PCIe Controller (x16)
/0/100/1/0                         display        TU117GLM [Quadro T1000 Mobile]
/0/100/2           /dev/fb0        display        CoffeeLake-H GT2 [UHD Graphics 630]
/0/100/4                           generic        Xeon E3-1200 v5/E3-1500 v5/6th Gen Core Processor Therm
/0/100/8                           generic        Xeon E3-1200 v5/v6 / E3-1500 v5 / 6th/7th/8th Gen Core 
/0/100/12                          generic        Cannon Lake PCH Thermal Controller
/0/100/14                          bus            Cannon Lake PCH USB 3.1 xHCI Host Controller
/0/100/14/0        usb1            bus            xHCI Host Controller
/0/100/14/0/4                      communication  AX200 Bluetooth
/0/100/14/0/7                      communication  Goodix Fingerprint Device
/0/100/14/0/c                      multimedia     Integrated_Webcam_HD
/0/100/14/1        usb2            bus            xHCI Host Controller
/0/100/14.2                        memory         RAM memory
...

# view SCSI, USB, IDE, PCI device data
lshw -businfo
Bus info          Device          Class          Description
============================================================
                                  system         Precision 5540 (0906)
                                  bus            07C17G
                                  memory         64KiB BIOS
                                  memory         32GiB System Memory
                                  memory         16GiB SODIMM DDR4 Synchronous 2667 MHz (0.4 ns)
                                  memory         16GiB SODIMM DDR4 Synchronous 2667 MHz (0.4 ns)
                                  memory         384KiB L1 cache
                                  memory         1536KiB L2 cache
                                  memory         12MiB L3 cache
cpu@0                             processor      Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
...

# view class info (get class from -short cmd)
lshw -class display
  *-display UNCLAIMED       
       description: 3D controller
       product: TU117GLM [Quadro T1000 Mobile]
       vendor: NVIDIA Corporation
       physical id: 0
       bus info: pci@0000:01:00.0

# view RAM information
lshw -class memory
  *-firmware                
       description: BIOS
       vendor: Dell Inc.
       physical id: 0
       version: 1.20.0
       date: 12/23/2022
       size: 64KiB
       capacity: 32MiB
       capabilities: pci pnp upgrade shadowing cdboot bootselect edd int13floppynec int13floppy1200 int13floppy720 int13floppy2880 int5printscreen int9keyboard int14serial int17printer acpi usb smartbattery biosbootspecification netboot uefi
  *-memory
       description: System Memory
       physical id: 37
       slot: System board or motherboard
       size: 32GiB
     *-bank:0
          description: SODIMM DDR4 Synchronous 2667 MHz (0.4 ns)
          product: AO1P26KCST2-BZISHC
          vendor: Fujitsu
          physical id: 0
          serial: EAB80200
          slot: DIMM A
          size: 16GiB
          width: 64 bits
          clock: 2667MHz (0.4ns)
     *-bank:1
     ...
  *-cache:0
       description: L1 cache
       physical id: 40
       slot: L1 Cache
       size: 384KiB
       capacity: 384KiB
       capabilities: synchronous internal write-back unified
       configuration: level=1
  *-cache:1
  ...
  *-memory UNCLAIMED
       description: RAM memory
       product: Cannon Lake PCH Shared SRAM
       vendor: Intel Corporation
       physical id: 14.2
       bus info: pci@0000:00:14.2
       version: 10
       width: 64 bits
       clock: 33MHz (30.3ns)
       capabilities: pm cap_list
       configuration: latency=0
       resources: memory:ed51e000-ed51ffff memory:ed526000-ed526fff
...
```

### Keyboard mappings

Check your distro mappings for correctness:

```bash
# RHEL
localectl
   System Locale: LANG=en_US.UTF-8
       VC Keymap: us
      X11 Layout: us

# view keymappings
localectl list-keymaps
ANSI-dvorak
al
al-plisi
amiga-de
amiga-us
applkey
at
at-mac

# change keymappings
localctl set-keymap <keymap-name>

# -------
# Debian
dpkg-reconfigure keyboard-configuration
```