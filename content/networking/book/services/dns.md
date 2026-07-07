+++
title = 'DNS'
date = '2026-06-30T00:08:08-04:00'
weight = 10
draft = false
+++

DNS translates the fully qualified domain names (FQDNs) that users type into applications at layer 7 into IP addresses used to route traffic at layers 3 and 4. DNS also maps an IP address back to an FQDN using a *pointer record* (PTR), called a *reverse lookup*.

The public DNS infrastructure consists of three tiers:

- *Root name servers*: 13 logical addresses labeled A through M (for example, `a.root-servers.net`) that anchor the DNS hierarchy. Each address is served by many physical servers distributed worldwide. They share the same IP address using anycast routing, so your query automatically reaches the nearest one. Root name servers hold the root zone, which maps each top-level domain to its authoritative name servers.
- *Upstream resolvers*: Recursive DNS servers operated by ISPs, cloud providers, and public services such as Google (8.8.8.8) and Cloudflare (1.1.1.1). These servers cache DNS responses and answer queries on behalf of clients and internal DNS servers.
- *Registrars*: Companies authorized to register domain names under a TLD (such as GoDaddy or Namecheap). When you register a domain, the registrar records your domain name and its authoritative name servers in the TLD registry, making the domain resolvable on the internet.

## Components

DNS is built from a small set of record types stored in zone files. Every configuration task and troubleshooting step involves reading or writing these records. Understanding what each one does is the foundation for everything else in this page.

| Record | Full name | Purpose |
|---|---|---|
| A | Address | Maps a hostname to an IPv4 address. The most common record type. |
| AAAA | IPv6 address | Maps a hostname to an IPv6 address. |
| CNAME | Canonical name | Creates an alias that points to another hostname instead of an IP address. |
| MX | Mail exchange | Identifies the mail server for a domain. Includes a priority number — lower wins. |
| NS | Name server | Lists the authoritative name servers for a zone. Required in every zone. |
| SOA | Start of Authority | The first record in every zone file. Defines the primary name server, admin contact, and timing values for zone transfers and caching. |
| PTR | Pointer | Maps an IP address back to a hostname. Used for reverse DNS lookups. |
| TXT | Text | Stores arbitrary text. Used for domain ownership verification, SPF email policy, and DKIM signing keys. |
| SRV | Service | Identifies the host and port for a specific service, such as SIP or LDAP. |

Zone files are plain text files stored on the authoritative name server. On Debian-based systems running BIND, they live in `/var/cache/bind/`. On RHEL-based systems, the default location is `/var/named/`. The `directory` directive in `named.conf.options` sets this path and tells BIND where to look when a zone block specifies a relative filename.

DNS uses UDP on port 53 for standard queries. It switches to TCP on port 53 for responses that exceed 512 bytes and for zone transfers between primary and secondary servers.

A CNAME differs from an A record in what it points to. An A record is the end of the chain — it maps a name directly to an IP address. A CNAME points to another name, which must eventually resolve to an A record through one or more additional lookups.

For example:

```bash
www      IN  A      192.168.1.20
ftp      IN  CNAME  www.coherentsecurity.com.
webmail  IN  CNAME  www.coherentsecurity.com.
```

Both `ftp.coherentsecurity.com` and `webmail.coherentsecurity.com` resolve to `192.168.1.20`, but they get there differently. `www` resolves in one step. `ftp` and `webmail` each resolve in two: first to `www.coherentsecurity.com`, then to `192.168.1.20`. Any number of CNAMEs can point to the same target. The practical benefit is that when the server's IP changes, you update only the A record for `www`. Every CNAME that points to it follows automatically because each points to the name, not the address.

## Features

### DNS recursion

*Recursion* is the ability of a DNS server to query other DNS servers on behalf of the client. When a server cannot answer a query from its own cache or zone file, it takes responsibility for working up the DNS hierarchy, querying root name servers, TLD servers, and authoritative name servers in turn, then returning the final answer to the client. The client sends one query and receives one answer. All intermediate lookups happen transparently.

Without recursion, the server returns a referral pointing the client to the next server in the chain, and the client makes each subsequent query itself. Recursion moves that work to the server, simplifying the client.

The DNS request flow section below demonstrates this: the internal DNS server and the forwarder each perform recursive queries until the authoritative answer is returned to the client.

### Forwarder entries

When a query cannot be answered from the internal zone file or cache, the internal DNS server forwards it to one or more upstream servers called *forwarders*. Configure at least two forwarders for redundancy. Upstream forwarders cache responses and expire them according to each record's TTL.

ISP DNS servers were historically the default forwarder choice. Larger public DNS providers have since become more reliable and offer additional features such as DNSSEC validation (cryptographic verification that a DNS response hasn't been tampered with in transit), privacy controls, and content filtering.

| Provider                 | Primary        | Secondary       |
| ------------------------ | -------------- | --------------- |
| Google                   | 8.8.8.8        | 8.8.4.4         |
| Cloudflare               | 1.1.1.1        | 1.0.0.1         |
| Quad9                    | 9.9.9.9        | 149.112.112.112 |
| OpenDNS / Cisco Umbrella | 208.67.222.222 | 208.67.220.220  |

### Caching

A DNS server caches responses from upstream servers and serves them directly for subsequent queries until the TTL expires. Adding memory to a DNS server increases the number of entries it can hold in cache. A larger cache means more queries are answered locally, which reduces round-trip latency, lowers load on upstream servers, and cuts external DNS traffic. It also improves resilience: if an upstream server is temporarily unavailable, the local DNS server continues answering from cache for any entry whose TTL has not yet expired.

### Dynamic registration

Servers typically use static IP addresses with DNS records added manually. Workstations often use DHCP (the protocol that automatically assigns IP addresses to devices on a network) and change addresses regularly. *Dynamic registration* keeps DNS current for these hosts through two mechanisms:

- The DHCP server updates DNS automatically when it assigns or renews a lease, adding or refreshing the host's A record with its current IP address.
- Hosts register themselves directly in DNS using the dynamic update protocol defined in RFC 2136.

Both mechanisms prevent DNS records for DHCP clients from becoming stale without requiring manual administrator updates.

### Host redundancy

DNS deployments typically include a *primary* server and one or more *secondary* servers. The primary holds the authoritative zone file. Secondary servers receive a copy through a *zone transfer*, a replication process that copies the zone file from the primary to the secondary in one direction.

The DNS server uses the serial number in the zone file's *Start of Authority (SOA)* record to determine when replication is needed. When the secondary checks in and finds the primary's serial number is higher than its own, it initiates a zone transfer to pull the updated zone file.

A secondary server protects against primary server failure and allows primary maintenance without a service interruption, since clients continue querying the secondary during that time.

## DNS request flow

A full DNS lookup for `www.example.com` (93.184.216.34) follows the path below. Once upstream servers have been queried and their answers are cached, most queries resolve at step 1 or step 2 without any external lookups.

1. The client checks its own local DNS cache for `www.example.com`. If the entry exists and its TTL has not expired, the client uses `93.184.216.34` immediately and sends no query.

2. The internal DNS server checks its cache for `www.example.com`. If the entry exists and its TTL has not expired, it returns `93.184.216.34` to the client. If the internal DNS server is *authoritative* for the queried domain (meaning it hosts the zone file for that domain), it returns the record directly from the zone file. This is an *authoritative answer*: the data comes from the zone file itself, not from a cached copy.

3. If the entry is not in cache or the TTL has expired, the internal DNS server forwards the query for `www.example.com` to an upstream *forwarder*. If the forwarder has `93.184.216.34` in its own cache, it returns the answer immediately. If the forwarder already holds a cached NS record pointing to the authoritative name server for `example.com`, it queries that server directly.

4. If the forwarder has no cached answer for `www.example.com`, it queries a root name server. There are 13 root name server addresses, labeled A through M. For example, `a.root-servers.net` (198.41.0.4) is one of the root servers at the top of the DNS hierarchy.

5. The root name server does not return `93.184.216.34`. It returns the authoritative name servers for the `.com` TLD (for example, `a.gtld-servers.net`).

6. The forwarder caches the TLD name server information and queries the `.com` TLD authoritative name server for `www.example.com`.

7. The `.com` authoritative name server returns the authoritative name servers for `example.com` (for example, `ns1.example.com`).

8. The forwarder queries `ns1.example.com` directly for `www.example.com`.

9. The authoritative name server for `example.com` returns the A record: `www.example.com` resolves to `93.184.216.34`.

10. The forwarder caches `93.184.216.34` and sends it to the internal DNS server.

11. The internal DNS server caches `93.184.216.34` and forwards it to the client. The client stores the answer in its local DNS cache and passes `93.184.216.34` to the application that made the request, such as a web browser.

## Internal DNS server

Most organizations run an internal DNS server for name resolution within their own network. The server hosts a zone file populated with DNS records for internal resources: servers, workstations, printers, and network devices that are not published on the public internet.

The zone file is populated by one or more of these methods:

- *Manual entry*: An administrator edits the zone file directly. Common for static infrastructure like servers and network devices.
- *Auto-registration*: Clients register their own DNS records when they join the network. Windows clients use dynamic DNS (DDNS) to update the server automatically when their IP address changes.
- *DHCP integration*: The DHCP server updates DNS records automatically when it assigns or renews a lease, keeping DNS synchronized with current IP address assignments.

When a client queries the internal DNS server for an internal hostname, the server answers immediately from its zone file or cache. The query stays inside the organization's network.

## Internet-facing DNS server

If you host your own domain, you implement an authoritative DNS server for one or more DNS zones. An internet-facing authoritative server fills the role in steps 8 and 9 of the DNS request flow above: a forwarder queries it directly, and it returns the actual record from its zone file.

Unlike an internal DNS server, an internet-facing server is exposed to the public internet. Security takes priority over performance in its design.

### Restrict recursion

An internet-facing authoritative server should never recurse. When recursion is enabled, the server queries upstream resolvers on behalf of any client to answer questions about domains it doesn't own. A server that does this is an *open resolver*.

Open resolvers are exploited in DNS amplification attacks. Restricting recursion limits the server to answering queries about its own zones only. Queries for external domains receive a REFUSED response.

### Cache is less important

An authoritative server holds the answer to every query it should receive in its zone files. Because it does not recurse, it has no upstream responses to cache. Memory goes toward holding zone data for fast lookups, not toward caching external records.

### Host redundancy

Adding a second authoritative server provides hardware redundancy, which matters more than adding cache to a single server. If you take the primary offline for maintenance, the secondary continues answering queries without interruption. For an internet-facing server, an unanswered query is a service outage for every user trying to reach your domain.

### Restrict zone transfers

A zone transfer copies your entire zone file from the primary server to a secondary. Transfers between your own servers keep them synchronized as you edit zone records. They should never be available to arbitrary clients.

An attacker who can request a zone transfer receives every hostname, IP address, and service in your zone in a single response, providing a complete map of your infrastructure. Restrict zone transfers by source IP so only your own secondary servers can initiate them. Individual DNS queries answer one question at a time; a zone transfer hands over everything at once.

### Rate limiting

Use *Response Rate Limiting (RRL)* to cap how many identical responses the server sends to a single source per time window.

DNS uses UDP. There is no handshake, only a request and a response. An attacker can send queries with a spoofed source IP, causing your server to send its responses to a victim instead of the actual sender. When the attacker uses many DNS servers simultaneously as reflectors, and DNS responses are larger than the queries that triggered them, the result is a *DNS amplification attack*: a small volume of attacker traffic generates a large flood directed at the victim.

RRL mitigates this by limiting responses to a source IP to a small number per time window. For example, allow 2 identical responses per source IP in a 5-minute window before throttling further responses.

Rate limiting also restricts reconnaissance. An attacker enumerating your zone by sending many varied queries is slowed significantly when each source address is individually limited.

### Restrict dynamic registration

Never allow dynamic registration on an internet-facing server unless you specifically offer DDNS as a service. An attacker who can register records in your zone can redirect traffic intended for your services or insert entries pointing to infrastructure they control.

## Common implementations

Most DNS deployments run one of two servers: BIND for full-featured authoritative and recursive use, or dnsmasq for lightweight forwarding on appliances and small networks.

## BIND

*BIND* (Berkeley Internet Name Domain), also called *named* after its daemon process, is the most widely deployed DNS server software on the internet. It runs on Linux and Unix systems and supports the full DNS feature set: authoritative zone hosting, recursive resolution, DNSSEC validation, zone transfers, views, and access control lists.

### Install BIND

The `bind9` package includes the BIND name server daemon (`named`) and its supporting utilities. Install it with:

```bash
sudo apt install -y bind9
```

After installation, BIND starts automatically and listens on port 53.

### Configure BIND

BIND's main configuration file is `/etc/bind/named.conf`. On Debian-based systems, this file contains only `include` directives that pull in three separate files:

```bash
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```

Splitting the configuration this way keeps each file focused on a single concern and makes edits easier to locate.

| File                       | Purpose                                                                                    |
| -------------------------- | ------------------------------------------------------------------------------------------ |
| `named.conf.options`       | Global server options: forwarders, recursion policy, listen addresses, and DNSSEC settings |
| `named.conf.local`         | Zone definitions for domains this server hosts or is authoritative for                     |
| `named.conf.default-zones` | Built-in zones: the root hints zone and localhost zones required by every DNS server       |

Edit `named.conf.options` to configure forwarders and recursion. Edit `named.conf.local` to add or modify zone definitions. Leave `named.conf.default-zones` unchanged unless you have a specific reason to override the defaults.

### Manage BIND

Use `systemctl` to control the `bind9` service:

```bash
sudo systemctl enable bind9                  # start automatically at boot
sudo systemctl start bind9                   # start the service now
sudo systemctl stop bind9                    # stop the service
sudo systemctl restart bind9                 # stop and restart — re-reads all config and zone files
sudo systemctl reload bind9                  # reload config without stopping — preferred for zone changes
sudo systemctl status bind9                  # show whether the service is running and recent log lines
```

Prefer `reload` over `restart` for zone file changes. `reload` applies the new config without dropping in-progress queries. `restart` stops the process entirely and briefly interrupts DNS service.

Validate configuration and zone files before reloading:

```bash
named-checkconf                                                                  # validate all config files
named-checkzone coherentsecurity.com /var/cache/bind/coherentsecurity.com.zone  # validate a zone file
```

Both commands exit silently with no output if the files are valid. Fix any errors they report before reloading — BIND will reject a bad config and continue running on the previous valid one.

#### View logs

View logs with `journalctl`:

```bash
journalctl -u bind9                        # all bind9 log entries
journalctl -u bind9 -f                     # follow log output in real time
journalctl -u bind9 -n 50                  # show the last 50 lines
journalctl -u bind9 --since "1 hour ago"   # entries from the last hour
```

### Syntax

BIND configuration files use a structured syntax similar to C:

- Each directive ends with `;`
- Blocks use `{ }` and close with `};`
- Lists of values inside a block are semicolon-separated
- String values are double-quoted
- Single-line comments use `//`; multi-line comments use `/* */`

```bash
options {
    directory "/var/cache/bind";        // string value, double-quoted
    forwarders { 8.8.8.8; 1.1.1.1; };  // list of values
    recursion yes;                      // keyword value
};
```

### Internal settings

An internal DNS server serves a known, trusted set of clients — employees, servers, and devices on your own network. Because you control who can reach it, you configure it differently than an internet-facing server. Recursion is enabled so clients can resolve external names through the server's forwarders. Queries are restricted to your internal address ranges to prevent outside hosts from using your server as an open resolver, which can be exploited in amplification attacks. Dynamic registration is allowed so workstations can register themselves as they join the network. The settings below reflect these priorities: full recursive resolution and convenient access for trusted clients, locked down to your internal network only.

#### named.conf.options

`named.conf.options` holds the global `options` block, which controls how the server behaves for all zones. A typical internal DNS server configuration:

```bash
options {
    directory "/var/cache/bind";
    listen-on port 53 { any; };
    listen-on-v6 { any; };
    allow-query { localhost; 192.168.0.0/16; 10.0.0.0/8; 172.16.0.0/12; };
    forwarders { 8.8.8.8; 8.8.4.4; 1.1.1.1; };
    recursion yes;
    dnssec-validation auto;
};
```

| Directive           | Purpose                                                                                                                                                                                              |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `directory`         | Working directory for BIND. Relative zone file paths resolve from here.                                                                                                                              |
| `listen-on port 53` | Interfaces and port BIND accepts queries on. `any` accepts queries on all interfaces. Restrict to a specific IP — for example, `{ 192.168.1.10; }` — to limit which interface the server listens on. |
| `listen-on-v6`      | Same as `listen-on` for IPv6. `any` accepts queries on all IPv6 interfaces.                                                                                                                          |
| `allow-query`       | ACL defining which clients may send queries. The three CIDR blocks cover the full RFC 1918 private address space.                                                                                    |
| `forwarders`        | Upstream resolvers BIND forwards queries to when it cannot answer locally.                                                                                                                           |
| `recursion`         | Enables recursive resolution. Combined with `allow-query`, recursion is available only to clients in the ACL.                                                                                        |
| `dnssec-validation` | Validates DNSSEC signatures on responses. `auto` loads the built-in root trust anchor so no manual key configuration is needed.                                                                      |

#### named.conf.local

`named.conf.local` holds zone definitions for domains this server hosts. Each `zone` block defines one domain and how the server handles it.

```bash
zone "coherentsecurity.com" IN {
    type master;
    file "coherentsecurity.com.zone";
    allow-update { 192.168.0.0/16; 10.0.0.0/8; 172.16.0.0/12; };
};
```

| Directive                        | Purpose                                                                                                                                                                                                                |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `zone "coherentsecurity.com" IN` | Declares the zone name. `IN` is the Internet class, which is the default for all standard DNS records and is typically omitted.                                                                                        |
| `type master`                    | This server is the primary authoritative server for the zone and holds the writable copy of the zone file. Secondary servers use `type slave` and pull the zone file from the primary.                                 |
| `file`                           | Path to the zone file containing DNS records for the domain. Relative paths resolve from the `directory` set in `named.conf.options` (`/var/cache/bind/` by default).                                                  |
| `allow-update`                   | ACL for dynamic DNS updates (RFC 2136). Clients matching these addresses can add or modify records in the zone without editing the zone file manually. The three blocks cover the full RFC 1918 private address space. |

BIND 9.11 introduced `primary` and `secondary` as aliases for `master` and `slave`. Both forms work in current versions.

#### Zone file

Create the zone file at `/var/cache/bind/coherentsecurity.com.zone`. BIND derives this path by combining two settings: the `directory "/var/cache/bind"` line in `named.conf.options` sets BIND's working directory, and the `file "coherentsecurity.com.zone"` line in `named.conf.local` gives the filename relative to that directory.

`/var/cache/bind` is the correct location for two reasons. First, it is BIND's designated working directory on Debian-based systems. Second, the `bind` user account that runs the `named` process has write access there. Write access is required when dynamic DNS updates are enabled: BIND writes a journal file alongside the zone file to track changes before committing them to disk.

```bash
$TTL 86400
@   IN  SOA ns1.coherentsecurity.com. admin.coherentsecurity.com. (
                2026070401  ; Serial (YYYYMMDDNN — increment on every change)
                3600        ; Refresh: how often secondaries check for updates
                900         ; Retry: how long a secondary waits before retrying a failed refresh
                604800      ; Expire: how long a secondary serves the zone if it cannot reach primary
                86400 )     ; Negative TTL: how long NXDOMAIN responses are cached

; Name servers
@       IN  NS      ns1.coherentsecurity.com.
@       IN  NS      ns2.coherentsecurity.com.

; A records
@       IN  A       192.168.1.10
ns1     IN  A       192.168.1.10
ns2     IN  A       192.168.1.11
www     IN  A       192.168.1.20
mail    IN  A       192.168.1.30

; Mail
@       IN  MX  10  mail.coherentsecurity.com.

; Alias
ftp     IN  CNAME   www.coherentsecurity.com.
```

| Section | Purpose                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `$TTL`  | Sets the default time-to-live in seconds for every record in the file. 86400 equals 24 hours. Resolvers cache each record for this long before re-querying. Set it lower if records change frequently; higher if they're stable.                                                                                                                                                                                                                                                                                              |
| SOA     | Required in every zone file. Exactly one per zone. Identifies the primary name server, the admin contact email (the first `.` replaces `@`), and five timing values that govern how secondary servers replicate and expire the zone. The serial number is the key field: incrementing it signals secondaries that the zone has changed and a zone transfer is needed.                                                                                                                                                         |
| NS      | Lists every authoritative name server for the zone. Both primary and secondary servers must appear here. Resolvers use these records to find which servers to query for this domain.                                                                                                                                                                                                                                                                                                                                          |
| A       | Maps a hostname to an IPv4 address. A *common name* is a short label relative to the zone origin. BIND appends the zone name automatically, so `www` becomes `www.coherentsecurity.com`. An *FQDN* includes the full name with a trailing dot: `www.coherentsecurity.com.` Both forms resolve to the same address. Omitting the trailing dot on a full name causes BIND to append the zone name a second time, producing `www.coherentsecurity.com.coherentsecurity.com`. The `@` entry maps the bare domain itself to an IP. |
| MX      | Identifies the mail server for the domain. The `10` is the priority; lower wins when multiple MX records exist. Mail servers for other domains look up this record to find where to deliver email addressed to `@coherentsecurity.com`.                                                                                                                                                                                                                                                                                       |
| CNAME   | Creates an alias. `ftp` resolves to whatever `www.coherentsecurity.com` resolves to. When the A record for `www` changes, `ftp` follows automatically. CNAMEs cannot coexist with other record types on the same name, so you cannot put a CNAME on `@`.                                                                                                                                                                                                                                                                      |

After editing the zone file, reload BIND to apply the changes:

```bash
systemctl reload bind9
```

#### ACLs

An *access control list (ACL)* in BIND is a named list of IP addresses or CIDR blocks. You define an ACL once and reference it by name wherever you need it. This keeps IP lists in one place — change them once and the update applies everywhere they are referenced.

To allow clients to register themselves in DNS so administrators can reach them by name rather than IP address, define ACLs for each client group and reference them in the zone's `allow-update` directive. Add this to `named.conf.local`:

```bash
acl dhcp-clients { 192.168.122.128/25; };
acl static-clients { 192.168.122.64/26; };

zone "coherentsecurity.com" {
    allow-update { dhcp-clients; static-clients; };
};
```

| Line                                             | Purpose                                                                                                                                                                                                        |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `acl dhcp-clients { 192.168.122.128/25; }`       | Defines an ACL named `dhcp-clients` covering addresses .128 through .255 in the `192.168.122.0/24` subnet. These are clients that receive their IP from DHCP and register their own DNS records automatically. |
| `acl static-clients { 192.168.122.64/26; }`      | Defines an ACL named `static-clients` covering addresses .64 through .127. These are clients with manually assigned IPs that also register in DNS.                                                             |
| `allow-update { dhcp-clients; static-clients; }` | Permits any address in either ACL to send a dynamic DNS update for the `coherentsecurity.com` zone. Addresses outside both ACLs are refused.                                                                   |

You could simplify this by allowing the entire subnet (`192.168.122.0/24`) or a corporate supernet (`10.0.0.0/8`), but broader rules let unintended devices register themselves: guest networks, IoT devices, and DMZ hosts would all be able to add records to your zone. Limit `allow-update` to the subnets where you actually want auto-registration.

When your ACL list grows, move the definitions into a separate file and pull it in with `include`. Create `/etc/bind/named.conf.acls` and place your ACL definitions there, then reference it in `named.conf`:

```bash
include "/etc/bind/named.conf.acls";
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```

ACLs must be defined before they are referenced, so the ACL include must appear before `named.conf.local` in the load order. This keeps ACL management in its own file, which is useful when a script or separate team member maintains the IP lists without touching zone or options configuration.

For example, the default `named.conf.local` zone block uses raw CIDR blocks in `allow-update`:

```bash
zone "coherentsecurity.com" IN {
    type master;
    file "coherentsecurity.com.zone";
    allow-update { 192.168.0.0/16; 10.0.0.0/8; 172.16.0.0/12; };
};
```

Define those ranges as named ACLs in `named.conf.acls`:

```bash
acl internal-clients {
    192.168.0.0/16;
    10.0.0.0/8;
    172.16.0.0/12;
};
```

Then replace the raw CIDRs in `named.conf.local` with the ACL name:

```bash
zone "coherentsecurity.com" IN {
    type master;
    file "coherentsecurity.com.zone";
    allow-update { internal-clients; };
};
```

If the same IP ranges appear in `allow-query` or other directives elsewhere in your config, they now all reference one definition. Update the ACL once and every directive that uses it reflects the change.

When making this change, update both files together. If you add the zone block to `named.conf` before updating `named.conf.local`, the same zone is defined in two places and BIND rejects it with a `type not present` error on the incomplete block. Remove or update the zone block in `named.conf.local` first, then add the ACL definitions and include to `named.conf`.

#### Verify the configuration

After reloading BIND, confirm the zone is being served and forwarding is working. Query the local server directly with `@127.0.0.1` so the test bypasses any other resolver on the system.

Check that internal zone records resolve:

```bash
dig @127.0.0.1 +short ns1.coherentsecurity.com A
192.168.1.10

dig @127.0.0.1 +short ns2.coherentsecurity.com A
192.168.1.11
```

- `@127.0.0.1` sends the query directly to the local BIND instance instead of the system's configured resolver.
- `+short` prints only the answer, not the full response headers.
- `ns1.coherentsecurity.com` is the name to look up.
- `A` requests an A record (IPv4 address).

Check that forwarding is working by resolving an external name:

```bash
dig @127.0.0.1 +short isc.sans.edu
104.20.34.201
172.66.151.107
```

`isc.sans.edu` is an external name that BIND cannot answer from its zone file. To resolve it, BIND forwards the query to the upstream resolvers defined in `named.conf.options`. A successful response confirms the forwarders are reachable and recursion is enabled.

If the internal queries return the correct IPs, BIND is loading and serving the zone file. If the external query returns IPs, the forwarders are reachable and recursion is working.

### Internet-facing settings

Historically, hosting your own domain required standing up a DNS server yourself or using one provided by your ISP, then managing zone files manually through the command line. Most organizations now host their DNS directly with their domain registrar instead. Major registrars such as Cloudflare, GoDaddy, and Namecheap operate their own global DNS infrastructure and give you a web UI to manage your zone records — adding, editing, and deleting entries without touching a config file.

This shifts the responsibility for infrastructure security and availability to the registrar. Your responsibility is protecting access to your registrar account. Enable multi-factor authentication (MFA) on the account to defend against *credential stuffing* — an attack where an adversary takes username and password combinations leaked from other data breaches and tries them against your registrar login. A successful login gives them full control over your DNS, letting them redirect your domain to infrastructure they control. MFA stops a stolen password from being enough.

If you do run your own internet-facing authoritative server, the settings below harden it for public exposure.

```bash
options {
    directory "/var/cache/bind";
    listen-on port 53 { any; };
    listen-on-v6 { any; };
    allow-query { any; };
    recursion no;
    dnssec-validation auto;
    rate-limit {
        responses-per-second 10;
        log-only yes;
    };
};
```

The key differences from an internal server: `allow-query` is open to any source, `recursion` is disabled, and there is no `forwarders` block. The `rate-limit` block is nested inside `options` because RRL is a server-wide behavior, not a per-zone setting.

#### Rate limiting DNS requests

Add a `rate-limit` block inside the `options` block in `/etc/bind/named.conf.options` to enable *Response Rate Limiting (RRL)*:

```bash
options {
    ...
    rate-limit {
        responses-per-second 10;
        log-only yes;
    };
};
```

| Directive | Purpose |
|---|---|
| `responses-per-second 10` | Caps the number of identical responses sent to a single source IP at 10 per second. Requests beyond that limit are dropped or truncated. |
| `log-only yes` | Test mode. BIND logs what it would rate-limit but does not drop any responses. Remove this line or set it to `no` to enforce the limit in production. |

#### Recursion and forwarding

An internet-facing authoritative server answers queries only for zones it owns. It should never recurse or forward — those features are for internal servers that resolve external names on behalf of clients. Leaving recursion enabled makes the server an open resolver that can be exploited in amplification attacks.

Remove the `forwarders` block from `named.conf.options` entirely and set recursion to off:

```bash
recursion no;
```

#### Allow query from any IP

An internal DNS server restricts `allow-query` to internal networks because it only serves internal clients. An internet-facing server must answer queries from anywhere on the internet.

Set `allow-query` to accept all source addresses:

```bash
allow-query { any; };
```

## dnsmasq

*dnsmasq* is a lightweight DNS forwarder and DHCP server designed for small networks and embedded systems. Its small footprint makes it a common choice for network appliances such as home routers, wireless access points, and perimeter firewalls. If a home network has a DNS server on its perimeter firewall or WAP, that server is likely running dnsmasq.

dnsmasq integrates DNS and DHCP directly: it reads the DHCP lease database and answers DNS queries from it, so workstations registered through DHCP are immediately resolvable by hostname without manual DNS entries. Network appliances that run dnsmasq typically provide a web GUI for reporting and configuration.

dnsmasq also supports blocklists. *Pi-hole* is an application built on dnsmasq that exposes this blocklist feature through a web interface, blocking ads and trackers at the DNS level for every device on the network.

## Glossary

Authoritative name server
: The DNS server that holds the definitive zone file for a domain and returns authoritative answers directly from that file, not from cache.

Cache
: A temporary store of DNS responses held by a resolver or client. Cached entries are reused until their TTL expires.

DNS amplification attack
: An attack in which an adversary spoofs a victim's IP address and sends small DNS queries to many open resolvers or authoritative servers. Each server sends its response to the victim, generating traffic that is larger than the original query. The ratio of response size to query size is the amplification factor.

FQDN
: Fully qualified domain name. The complete host address including all labels through the root: `www.example.com.`

Forwarder
: An upstream DNS server to which an internal DNS server sends queries it cannot answer locally.

Open resolver
: A DNS server that answers recursive queries from any source on the internet. Open resolvers can be abused as reflectors in amplification attacks.

PTR record
: A pointer record that maps an IP address to an FQDN, used for reverse DNS lookups.

Recursive resolver
: A DNS server that performs the full lookup chain on behalf of a client, querying multiple servers until it returns an authoritative answer.

Registrar
: A company authorized to register domain names under a TLD. The registrar submits name server records to the TLD registry when you register a domain.

Root name server
: One of 13 logical DNS servers (A through M) at the top of the DNS hierarchy. Root name servers return the authoritative name servers for each TLD.

RRL
: Response Rate Limiting. A DNS server feature that caps how many identical responses the server sends to a single source IP within a given time window, mitigating amplification attacks and reconnaissance.

TLD
: Top-level domain. The rightmost label in a domain name: `.com`, `.org`, `.net`, `.gov`.

TTL
: Time to live. The number of seconds a DNS record may be held in cache before it must be refreshed from the authoritative source.

Zone
: A portion of the DNS namespace that a single authoritative name server is responsible for. A zone contains all the DNS records for one domain. For example, `coherentsecurity.com` is a zone that includes records for `www.coherentsecurity.com`, `mail.coherentsecurity.com`, and any other hostnames under that domain.

Zone file
: A text file on an authoritative name server containing the DNS records for a domain: A, AAAA, CNAME, MX, PTR, NS, SOA, and others.
