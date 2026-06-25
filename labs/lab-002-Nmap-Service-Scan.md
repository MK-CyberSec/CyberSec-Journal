# Lab 002 — Nmap Service Scan
**Date:** June 17, 2026  
**Status:** Complete

---

## Objective
Perform a service version scan against the Ubuntu Server VM using 
Nmap to identify open ports and the services running on them.

---

## Tools Used
- Kali Linux 2026.1
- Nmap
- Ubuntu Server 26.04 LTS (target machine)

---

## What I Did
Ran a service detection scan from Kali against the Ubuntu Server VM 
to enumerate open ports and identify the specific service and version 
running on each. This is a fundamental reconnaissance technique used 
to understand what is exposed on a target system before any further 
testing or hardening can take place.

---

## Steps Taken
1. Confirmed Ubuntu VM was powered on and noted its IP address using `ip a`
2. Opened a terminal on the Kali VM
3. Ran the command: `nmap -sV [Ubuntu ip]`
4. Reviewed the output, identifying each open port and its associated service
5. Cross-referenced unfamiliar services against notes from Professor Messer and TryHackMe

---

## Output
Starting Nmap 7.98 ( https://nmap.org ) at 2026-06-24 18:16 -0400
Stats: 0:00:03 elapsed; 0 hosts completed (0 up), 1 undergoing ARP Ping Scan
ARP Ping Scan Timing: About 100.00% done; ETC: 18:16 (0:00:00 remaining)
Nmap scan report for 192.168.56.106
Host is up (0.00058s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.2p1 Ubuntu 2ubuntu3.2 (Ubuntu Linux; protocol 2.0)
MAC Address: 08:00:27:AD:07:D7 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 4.96 seconds

---

## Issues Encountered
No issues encountered during 
this scan.

---

## What This Means
The `-sV` flag instructs Nmap to attempt service version detection 
on every open port it finds, rather than just reporting that a port 
is open. This is more useful than a basic scan because knowing the 
exact service and version running allows a security professional to 
identify outdated software, known vulnerabilities tied to specific 
versions, and unnecessary services that should be closed or restricted.

---

## What I Learned
- How to perform a service version scan using Nmap
- The difference between a basic port scan and a service detection scan
- How to interpret Nmap output — port number, state, service name, version
- Why identifying running services is a foundational step in vulnerability assessment

---

## Next Steps
- Test how firewall configuration changes affect scan results (Lab 003)
- Begin researching any vulnerabilities tied to identified service versions
