+++
title = 'Services'
date = '2026-06-30T00:08:08-04:00'
weight = 60
draft = false
+++

## DNS

DNS translates the fully qualified domain names (FQDNs) that users type into applications at layer 7 into IP addresses used to route traffic at layers 3 and 4. DNS also maps an IP address back to an FQDN using a *pointer record* (PTR), called a *reverse lookup*.

The public DNS infrastructure consists of three tiers:

- *Root name servers*: 13 logical addresses labeled A through M (for example, `a.root-servers.net`) that anchor the DNS hierarchy. Each address is served by many physical servers distributed worldwide using anycast routing. Root name servers hold the root zone, which maps each top-level domain to its authoritative name servers.
- *Upstream resolvers*: Recursive DNS servers operated by ISPs, cloud providers, and public services such as Google (8.8.8.8) and Cloudflare (1.1.1.1). These servers cache DNS responses and answer queries on behalf of clients and internal DNS servers.
- *Registrars*: Companies authorized to register domain names under a TLD (such as GoDaddy or Namecheap). When you register a domain, the registrar records your domain name and its authoritative name servers in the TLD registry, making the domain resolvable on the internet.

### Terms

Authoritative name server
: The DNS server that holds the definitive zone file for a domain and returns authoritative answers directly from that file, not from cache.

DNS amplification attack
: An attack in which an adversary spoofs a victim's IP address and sends small DNS queries to many open resolvers or authoritative servers. Each server sends its response to the victim, generating traffic that is larger than the original query. The ratio of response size to query size is the amplification factor.

Open resolver
: A DNS server that answers recursive queries from any source on the internet. Open resolvers can be abused as reflectors in amplification attacks.

RRL
: Response Rate Limiting. A DNS server feature that caps how many identical responses the server sends to a single source IP within a given time window, mitigating amplification attacks and reconnaissance.

Cache
: A temporary store of DNS responses held by a resolver or client. Cached entries are reused until their TTL expires.

FQDN
: Fully qualified domain name. The complete host address including all labels through the root: `www.example.com.`

Forwarder
: An upstream DNS server to which an internal DNS server sends queries it cannot answer locally.

PTR record
: A pointer record that maps an IP address to an FQDN, used for reverse DNS lookups.

Registrar
: A company authorized to register domain names under a TLD. The registrar submits name server records to the TLD registry when you register a domain.

Recursive resolver
: A DNS server that performs the full lookup chain on behalf of a client, querying multiple servers until it returns an authoritative answer.

Root name server
: One of 13 logical DNS servers (A through M) at the top of the DNS hierarchy. Root name servers return the authoritative name servers for each TLD.

TLD
: Top-level domain. The rightmost label in a domain name: `.com`, `.org`, `.net`, `.gov`.

TTL
: Time to live. The number of seconds a DNS record may be held in cache before it must be refreshed from the authoritative source.

Zone file
: A text file on an authoritative name server containing the DNS records for a domain: A, AAAA, CNAME, MX, PTR, NS, SOA, and others.

### Features

#### DNS recursion

*Recursion* is the ability of a DNS server to query other DNS servers on behalf of the client. When a server cannot answer a query from its own cache or zone file, it takes responsibility for working up the DNS hierarchy, querying root name servers, TLD servers, and authoritative name servers in turn, then returning the final answer to the client. The client sends one query and receives one answer. All intermediate lookups happen transparently.

Without recursion, the server returns a referral pointing the client to the next server in the chain, and the client makes each subsequent query itself. Recursion moves that work to the server, simplifying the client.

The request flow described earlier uses recursive resolution: the internal DNS server and the forwarder each perform recursive queries until the authoritative answer is returned to the client.

#### Forwarder entries

When a query cannot be answered from the internal zone file or cache, the internal DNS server forwards it to one or more upstream servers called *forwarders*. Configure at least two forwarders for redundancy. Upstream forwarders cache responses and expire them according to each record's TTL.

ISP DNS servers were historically the default forwarder choice. Larger public DNS providers have since become more reliable and offer additional features such as DNSSEC validation, privacy controls, and content filtering.

| Provider | Primary | Secondary |
|---|---|---|
| Google | 8.8.8.8 | 8.8.4.4 |
| Cloudflare | 1.1.1.1 | 1.0.0.1 |
| Quad9 | 9.9.9.9 | 149.112.112.112 |
| OpenDNS / Cisco Umbrella | 208.67.222.222 | 208.67.220.220 |

#### Caching

A DNS server caches responses from upstream servers and serves them directly for subsequent queries until the TTL expires. Adding memory to a DNS server increases the number of entries it can hold in cache. A larger cache means more queries are answered locally, which reduces round-trip latency, lowers load on upstream servers, and cuts external DNS traffic. It also improves resilience: if an upstream server is temporarily unavailable, the local DNS server continues answering from cache for any entry whose TTL has not yet expired.

#### Dynamic registration

Servers typically use static IP addresses with DNS records added manually. Workstations often use DHCP and change addresses regularly. *Dynamic registration* keeps DNS current for these hosts through two mechanisms:

- The DHCP server updates DNS automatically when it assigns or renews a lease, adding or refreshing the host's A record with its current IP address.
- Hosts register themselves directly in DNS using the dynamic update protocol defined in RFC 2136.

Both mechanisms prevent DNS records for DHCP clients from becoming stale without requiring manual administrator updates.

#### Host redundancy

DNS deployments typically include a *primary* server and one or more *secondary* servers. The primary holds the authoritative zone file. Secondary servers receive a copy through a *zone transfer*, a replication process that copies the zone file from the primary to the secondary in one direction.

The DNS server uses the serial number in the zone file's *Start of Authority (SOA)* record to determine when replication is needed. When the secondary checks in and finds the primary's serial number is higher than its own, it initiates a zone transfer to pull the updated zone file.

A secondary server protects against primary server failure and allows primary maintenance without a service interruption, since clients continue querying the secondary during that time.

### DNS request flow

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

Use *RRL* to cap how many identical responses the server sends to a single source per time window.

DNS uses UDP. There is no handshake, only a request and a response. An attacker can send queries with a spoofed source IP, causing your server to send its responses to a victim instead of the actual sender. When the attacker uses many DNS servers simultaneously as reflectors, and DNS responses are larger than the queries that triggered them, the result is a *DNS amplification attack*: a small volume of attacker traffic generates a large flood directed at the victim.

RRL mitigates this by limiting responses to a source IP to a small number per time window. For example, allow 2 identical responses per source IP in a 5-minute window before throttling further responses.

Rate limiting also restricts reconnaissance. An attacker enumerating your zone by sending many varied queries is slowed significantly when each source address is individually limited.

### Restrict dynamic registration

Never allow dynamic registration on an internet-facing server unless you specifically offer DDNS as a service. An attacker who can register records in your zone can redirect traffic intended for your services or insert entries pointing to infrastructure they control.
