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

Character
: Transfers data one char at a time. Often used for serial devices such as terminals or USB devices

Block
: Transfers large blocks of data. Used for high-speed data transfer devices like hard drives or network cards.


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