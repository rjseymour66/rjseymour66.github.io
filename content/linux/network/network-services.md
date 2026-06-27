+++
title = 'Network Services'
date = '2025-09-07T19:03:14-04:00'
weight = 20
draft = false
+++


## Network addressing

In the network `192.168.1.0/24`, two addresses have reserved purposes and cannot be assigned to hosts:

| Address         | Purpose                                                       |
| :-------------- | :------------------------------------------------------------ |
| `192.168.1.0`   | Network address. Identifies the network itself.               |
| `192.168.1.255` | Broadcast address. Sends packets to all hosts on the network. |

A **DHCP reservation** (also called a static lease) assigns the same address to a specific device each time it connects. Use reservations for devices that need a predictable address, such as servers or admin workstations.

## DHCP server

The DHCP server assigns IP addresses to clients on your network. The server itself must
have a static IP address. The default configuration file is `/etc/dhcp/dhcpd.conf`.
Lease assignments are recorded in `/var/lib/dhcp/dhcpd.leases` and logged to `/var/log/syslog`.

### Install DHCP

Install and configure the DHCP server when you want to manage address assignment on your
local network directly. This is useful in a home lab or small office where you need control
over lease times, DNS settings, or address reservations.

1. Install the package.
   ```bash
   apt install isc-dhcp-server
   ```
2. Stop the service during initial configuration.
   ```bash
   systemctl stop isc-dhcp-server
   ```
3. Disable the IPv6 service if you are not using it.
   ```bash
   systemctl stop isc-dhcp-server6
   systemctl disable isc-dhcp-server6
   ```
4. Back up the default configuration file.
   ```bash
   mv /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.orig
   ```
5. Create the configuration file.
   ```bash
   vim /etc/dhcp/dhcpd.conf
   ```
6. Assign the network interface in `/etc/default/isc-dhcp-server`. Set `INTERFACESv4`
   to the name of your network interface:
   ```bash
   INTERFACESv4="enp0s3"
   ```
7. Start the service and verify it is running.
   ```bash
   systemctl start isc-dhcp-server
   systemctl status isc-dhcp-server
   ```

To view logs and check lease assignments:

```bash
tail -f /var/log/syslog             # view logs in real time
cat /var/lib/dhcp/dhcpd.leases      # check lease assignments
```

### Configuration file

#### /etc/dhcp/dhcpd.conf

Configure lease times, network options, and the client address range. Set `authoritative`
if this is the only DHCP server on the network:

```bash
default-lease-time 43200;                   # clients must renew their lease every 43200 seconds (12 hours)
max-lease-time 86400;                       # maximum lease duration in seconds (1 day)
option subnet-mask 255.255.255.0;           # subnet mask sent to clients
option broadcast-address 192.168.1.255;     # broadcast address sent to clients
option domain-name "local.lan";             # domain name assigned to clients (<hostname>.local.lan)
authoritative;                              # declares this server as authoritative for this network
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.1 192.168.1.240;        # address range available to clients
    option routers 192.168.1.1;             # default gateway sent to clients
    option domain-name-servers 192.168.1.1; # DNS server address sent to clients
}
```

## DNS server

A DNS server resolves domain names to IP addresses. Organizations often run a local DNS
server to resolve hostnames within their network. If the local server cannot resolve a
name, it forwards the request to an external DNS server.

Linux DNS servers use the Berkeley Internet Name Domain (BIND) package. The main
configuration file is `/etc/bind/named.conf`, which links to additional configuration
files. The most basic use of BIND is as a **caching name server**: it forwards queries
to an external DNS server and caches the responses. Subsequent lookups for the same
name are served from the cache, reducing latency.

Install the package and verify the current DNS server address:

```bash
apt install bind9
resolvectl              # shows the IP address of the active DNS server
```

### Caching name server

A caching name server forwards DNS queries to an external server and caches the responses.
Repeated lookups are served from the cache without contacting the external server.

1. Edit the BIND options file.
   ```bash
   vim /etc/bind/named.conf.options
   ```
   Add your forwarding DNS servers inside the `forwarders` block:
   ```bash
   options {
       ...
       forwarders {
           8.8.8.8;
           8.8.4.4;
       };
       ...
   };
   ```
2. Restart the service and verify it is running.
   ```bash
   systemctl restart bind9
   systemctl status bind9
   ```
3. Update the DHCP configuration so new clients receive the local DNS server address.
   Edit `/etc/dhcp/dhcpd.conf` and set `domain-name-servers` in the subnet block:
   ```bash
   subnet 192.168.1.0 netmask 255.255.255.0 {
       ...
       option domain-name-servers <dns-addr>;
   }
   ```
4. Install the DNS test utilities.
   ```bash
   apt install dnsutils
   ```
5. Verify caching is working by running the same query twice. The second query time
   should drop to 0 milliseconds:
   ```bash
   dig www.mlb.com
   ```
   First query:
   ```bash
   ;; Query time: 15 msec
   ```
   Second query:
   ```bash
   ;; Query time: 0 msec
   ```

### Internal DNS

An internal DNS server resolves hostnames within your local network. Configure it by
defining a zone in `/etc/bind/named.conf.local` and creating a zone file that lists your
hosts and their IP addresses:

1. Edit the local zone configuration file.
   ```bash
   vim /etc/bind/named.conf.local
   ```
   Define the zone name and the path to its zone file:
   ```bash
   zone "local.lan" IN {
       type master;
       file "/etc/bind/net.local.lan";
   };
   ```
2. Create the zone file.
   ```bash
   vim /etc/bind/net.local.lan
   ```
3. Restart the service and check for errors.
   ```bash
   systemctl restart bind9
   systemctl status bind9
   ```

#### /etc/bind/net.local.lan

The zone file defines DNS records for your local network. `@` represents the zone name
(`local.lan`). Increment the serial number after each change so BIND detects updates.
The recommended serial format is `YYYYMMDDNN`:

```bash
$TTL 1D                                             # record TTL: how long clients cache responses
@ IN SOA local.lan. hostmaster.local.lan. (         # Start of Authority: declares this server authoritative for local.lan
                                                    # hostmaster.local.lan is the administrator contact address
    202401131;  serial                              # increment after each change (format: YYYYMMDDNN)
    8H ;        refresh                             # secondary servers check for updates every 8 hours
    4H ;        retry                               # secondary servers retry after 4 hours on error
    4W ;        expire                              # zone expires after 4 weeks if the primary is unreachable
    1D )        minimum                             # minimum TTL for negative responses
IN A 192.168.1.1                                    # name server IP address
;
@ IN NS hermes.local.lan.                           # name server hostname
fileserv    IN  A       192.168.1.3                 # local host records
hermes      IN  A       192.168.1.1                 # name server host record (required or resolution may fail)
mailserv    IN  A       192.168.1.5
mail        IN  CNAME   mailserv.                   # CNAME: alias pointing to mailserv
web01       IN  A       192.168.1.7
```

### Internet gateway setup

A gateway routes traffic between your local network and an external network such as the
internet. Administrators often run DNS, DHCP, and routing on the same server. A gateway
server requires two network interfaces: one connected to the ISP device using DHCP, and
one connected to the internal network switch.

Enable IP forwarding temporarily until the next reboot:

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```

To make IP forwarding persist across reboots, uncomment the following line in
`/etc/sysctl.conf`:

```bash
net.ipv4.ip_forward=1
```

## OpenSSH

OpenSSH lets you open a command shell on a remote server. It has two components: the
server daemon (`sshd`) and the client (`ssh`). Most distributions install OpenSSH by
default and configure it to start automatically. It listens on port 22.

You have the same permissions as the user you connect as. When you end a session, all
background processes started during that session are terminated.

Manage the service with these commands:

```bash
apt install openssh-client          # install the client
apt install openssh-server          # install the server
systemctl status ssh                # verify sshd is running
systemctl start ssh                 # start the service
systemctl enable ssh                # enable the service to start at boot
ss -tunlp | grep ssh                # verify sshd is listening for connections
```

Connect to a remote server:

```bash
ssh 10.20.30.40                     # connect as the current user on port 22
ssh <username>@<addr>               # connect as <username> on port 22
ssh -p 2242 <username>@<addr>       # connect on a non-default port
exit                                # end the SSH session
```

### SSH keys

[Ubuntu docs](https://help.ubuntu.com/community/SSH/OpenSSH/Keys)

By default, SSH authenticates with a password. Public key authentication is more secure
because your password is never transmitted. You generate a mathematically linked key
pair: a private key you keep on your local machine and a public key you copy to remote
servers. When you connect, SSH verifies your identity by matching your private key
against the public key on the server.

A passphrase on your private key is optional but recommended. Use `ssh-agent` to cache
the passphrase so you only enter it once per session.

Key file permissions are set automatically: the public key is readable by all, and the
private key is readable only by its owner.

#### Generate a key pair

Generate a key pair with `ssh-keygen`:

```bash
ssh-keygen                          # generate an ed25519 key pair (default)
ssh-keygen -t rsa -b 4096           # generate an RSA key pair
```

`ssh-keygen` defaults to *Ed25519*, an elliptic curve algorithm with a fixed 256-bit key. Ed25519 is faster to generate, faster to verify, and produces shorter key material than RSA. Use it for any modern system running OpenSSH 6.5 or later (released 2014).

Use the `-t rsa -b 4096` options to generate an *RSA* key at 4096 bits. RSA is supported by virtually every SSH implementation, including legacy systems and some enterprise environments that mandate RSA by policy. Use it when connecting to older infrastructure or when a compliance requirement specifies RSA explicitly. For everything else, Ed25519 is the better choice.



#### Add a key to ssh-agent

`ssh-agent` caches your passphrase so you only enter it once per session:

```bash
eval $(ssh-agent)                                   # start ssh-agent if not already running
ssh-add ~/.ssh/id_rsa                               # add the key (prompts for passphrase)
```

#### Change the passphrase on an existing key

Run `ssh-keygen -p` to update the passphrase on a key you have already generated:

```bash
ssh-keygen -p
```

### Copy to a remote server

`ssh-copy-id` copies your public key to a remote server's `authorized_keys` file. It
creates `~/.ssh/` and `authorized_keys` on the remote server if they do not exist.
Additional keys are appended to the file, one per line:

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub <user>@<ip-addr>
```

### SSH config file

The SSH client reads `~/.ssh/config` on each connection. The file is not created by
default. Each `Host` entry defines a shorthand alias with a hostname, user, and port.
Use an IP address for `HostName` if the server is not listed in `/etc/hosts`.

See all available options:

```bash
man ssh_config
```

Each entry follows this format:

```bash
Host <alias>                                # shorthand name you use with the ssh command
    HostName <ip-address-or-hostname>       # IP address or hostname of the remote server
    User <username>                         # user to connect as
    Port <port>                             # SSH port (default is 22)
```

Example:

```bash
Host u24
    HostName 10.20.30.40
    User linuxuser1
    Port 22
```

### Configuring

Default sshd settings do not meet the hardening recommendations in the [CIS Benchmarks](/networking/book/security/#benchmarks). Before making changes, use `sshd -T` to print the full effective configuration — every setting sshd is currently using, including defaults that don't appear in `sshd_config`:

```bash
sudo sshd -T                          # print the full effective sshd configuration
sudo sshd -T | grep passwordauth      # find a specific setting
sudo sshd -T | grep permitrootlogin   # check root login status
```

`sshd -T` reflects what the daemon is actually running with, not just what is uncommented in the config file. This makes it the reliable source for verifying that a change in `/etc/ssh/sshd_config` took effect after a reload.

The settings most commonly flagged by CIS Benchmarks are:

- *Root login*: disable direct root access over SSH.
- *Logging level*: set `LogLevel VERBOSE` to capture key fingerprints and authentication events.
- *Key exchange and MAC algorithms*: restrict to modern algorithms and remove weak or deprecated ones.
- *Idle timeout*: set `ClientAliveInterval` and `ClientAliveCountMax` to disconnect inactive sessions automatically.
- *MaxSessions*: limit the number of concurrent sessions per connection to reduce exposure from compromised credentials.

Edit `/etc/ssh/sshd_config` to apply hardening changes, then reload the service:

```bash
sudo vim /etc/ssh/sshd_config
sudo systemctl reload ssh
```

#### Root login

Permitting root login over SSH violates the *principle of least privilege*: root has unrestricted access to every file, process, and service on the system. A successful brute-force attack or stolen credential against root gives an attacker immediate full control with no further escalation required. Disabling root login forces all access through named user accounts, which creates an audit trail and limits blast radius if a credential is compromised.

The default value on many distributions is `prohibit-password`, which blocks password authentication for root but still allows key-based root login. Set it to `no` to block root login entirely.

1. Check the current setting.
   ```bash
   sshd -T | grep permitroot
   permitrootlogin without-password
   ```
2. Edit `sshd_config` and set `PermitRootLogin no`.
   ```bash
   vim /etc/ssh/sshd_config


   # Authentication:

   #LoginGraceTime 2m
   #PermitRootLogin prohibit-password
   PermitRootLogin no
   #StrictModes yes
   #MaxAuthTries 6
   #MaxSessions 10
   ```
3. Verify the change took effect.
   ```bash
   sshd -T | grep permitroot
   ```
   ```
   permitrootlogin no
   ```

#### Ciphers

SSH supports multiple encryption algorithms for the session cipher. Older algorithms — including `3des-cbc`, `arcfour`, and CBC-mode AES — have known weaknesses and should not be accepted. Restricting sshd to CTR-mode and GCM-mode AES removes the vulnerable options without breaking compatibility with modern clients.

1. Check the currently advertised ciphers.
   ```bash
   sshd -T | grep ciphers

   ciphers chacha20-poly1305@openssh.com,aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com
   ```
2. From a remote host, use nmap to enumerate all algorithms the server advertises.
   ```bash
   nmap -p22 -Pn --open 192.168.122.201 --script ssh2-enum-algos.nse

   PORT   STATE SERVICE
   22/tcp open  ssh
   | ssh2-enum-algos:
   |   kex_algorithms: (12)
   ...
   |   encryption_algorithms: (6)
   |       chacha20-poly1305@openssh.com
   |       aes128-ctr
   |       aes192-ctr
   |       aes256-ctr
   |       aes128-gcm@openssh.com
   |       aes256-gcm@openssh.com
   ...
   |_      zlib@openssh.com
   MAC Address: 52:54:00:56:D1:6B (QEMU virtual NIC)
   ```
3. Edit `sshd_config` and set the `Ciphers` directive.
   ```bash
   vim /etc/ssh/sshd_config
   
   # Ciphers and keying
   Ciphers aes256-ctr,aes192-ctr,aes128-ctr
   ```
4. Reload the service.
   ```bash
   systemctl reload ssh
   ```
5. Verify the change took effect.
   ```bash
   sshd -T | grep ciphers
   
   ciphers aes256-ctr,aes192-ctr,aes128-ctr
   ```

### Troubleshooting

Connection refused means the host is reachable but sshd isn't listening. No route to host or timeout means a network or firewall problem.

#### Diagnose the connection

Use `-v` for verbose output to see where the connection fails:

```bash
ssh -v user@host        # verbose
ssh -vvv user@host      # maximum verbosity
```

Check that port 22 is open on the remote host:

```bash
nc -zv <host> 22
```

#### Check sshd on the remote host

If you get a connection refused error, sshd isn't running or isn't listening on port 22. This happens after a fresh install, after a reboot if the service isn't enabled, or if sshd crashed.

```bash
systemctl status ssh            # Debian/Ubuntu use 'ssh', not 'sshd'
systemctl start ssh
ss -tlnp | grep 22              # confirm sshd is listening
journalctl -u ssh -n 50        # recent logs
```

#### Check the firewall

A firewall blocking port 22 produces a timeout or no route to host error. Check this when the host is reachable by ping but SSH hangs without a connection refused message.

```bash
ufw status                              # ufw (Ubuntu)
firewall-cmd --list-services            # firewalld (RHEL/Fedora)
iptables -L INPUT -n | grep 22         # iptables
```

#### Fix key permission errors

SSH refuses keys with overly permissive file modes:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_rsa
```

#### Clear a stale host key

SSH stores each host's key fingerprint in `~/.ssh/known_hosts` on first connect. If the remote host's key changes because the OS was reinstalled, the VM was recreated, or a new VM reused the same IP, SSH blocks the connection with a "host identification has changed" error. Remove the stale entry and reconnect:

```bash
ssh-keygen -R <host-ip>
```
