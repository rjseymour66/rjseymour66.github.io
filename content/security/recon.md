+++
title = 'Reconnaissance'
date = '2026-05-09T11:27:34-04:00'
weight = 10
draft = false
+++

Reconnaissance is the first phase of any attack or security assessment. Before exploiting a system, an attacker gathers as much information as possible about the target: its network layout, exposed services, software versions, and the people who manage it. The more an attacker knows, the more precisely they can strike.

Security professionals perform the same reconnaissance. Understanding what an attacker can discover about your systems lets you reduce your exposure before someone takes advantage of it.

## Passive vs. active

Reconnaissance falls into two categories:

*Passive reconnaissance* collects information without interacting directly with the target. You gather data from public sources: DNS records, WHOIS data, job postings, social media profiles, and archived web pages. The target never sees you.

*Active reconnaissance* interacts directly with the target. Port scans, banner grabs, and vulnerability probes send packets to the target's systems. This approach yields richer data but generates logs and may trigger alerts.

## What reconnaissance reveals

A thorough reconnaissance phase surfaces:

- Open ports and running services
- Software versions and known vulnerabilities
- Network topology and IP ranges
- Employee names, roles, and email formats
- Domain structure and subdomains
- Technologies in use (web frameworks, CMS, mail servers)

Defenders use this same lens to audit their own attack surface. If you can find it, an attacker can too.

## Subdomains in Kali