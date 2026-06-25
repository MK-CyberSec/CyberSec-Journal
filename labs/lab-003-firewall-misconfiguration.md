# Lab 003 — Firewall Misconfiguration
**Date:** June 17, 2026  
**Status:** Complete

---

## Objective
Demonstrate the security impact of firewall misconfiguration by 
disabling the firewall on the Ubuntu Server VM, scanning with Nmap 
to observe exposed services, then re-enabling the firewall and 
scanning again to confirm reduced exposure.

---

## Tools Used
- Kali Linux 2026.1
- Nmap
- Ubuntu Server 26.04 LTS (target machine)
- UFW (Uncomplicated Firewall)

---

## What I Did
Intentionally disabled the firewall on the Ubuntu target machine to 
observe how the attack surface changes when no firewall rules are 
enforced, then re-enabled it and rescanned to confirm the firewall 
was actively reducing exposed services. This simulates a real-world 
misconfiguration scenario and demonstrates the practical value of 
firewall enforcement.

---

## Steps Taken
1. Confirmed baseline state by running `sudo ufw status` on Ubuntu
2. Disabled the firewall: `sudo ufw disable`
3. Switched to Kali and ran: `nmap -sV [Ubuntu IP]`
4. Recorded all open ports and services visible with firewall disabled
5. Returned to Ubuntu and re-enabled the firewall: `sudo ufw enable`
6. Switched back to Kali and ran the same scan again: `nmap -sV [Ubuntu IP]`
7. Compared both sets of results

---

## Output

**Scan with firewall disabled:**
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

**Scan with firewall enabled:**
Starting Nmap 7.98 ( https://nmap.org ) at 2026-06-24 18:16 -0400
Nmap scan report for 192.168.56.106
Host is up (0.00089s latency).
All 1000 scanned ports on 192.168.56.106 are in ignored states.
Not shown: 1000 filtered tcp ports (no-response)
MAC Address: 08:00:27:AD:07:D7 (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.47 seconds

---

## Issues Encountered
After rebuilding the Ubuntu VM, the network adapter 
reset to its default setting and lost the Host-Only configuration, 
causing the VM to fall back to its loopback address only. 
This caused an initial Nmap scan to hang indefinitely against an unreachable target. 
Resolved by manually resetting the network adapter to Host-Only mode in 
VirtualBox VM settings and confirming the new IP address before rescanning.

---

## What This Means
A firewall acts as a gatekeeper, controlling which ports and services 
are reachable from outside the machine. When disabled, every running 
service becomes visible and potentially accessible to anyone who can 
reach the machine on the network. This lab demonstrates why firewall 
configuration is a critical baseline security control — without it, 
the actual attack surface of a system is significantly larger than 
intended.

With the firewall disabled, Nmap identified port 22 as open and successfully 
fingerprinted the exact SSH service and version running. With the firewall enabled, 
the same scan returned no usable information — every port appeared as 'filtered' rather 
than 'closed,' meaning the firewall was silently dropping packets rather than actively rejecting them. 
This is a more effective security posture than simply closing a port, because it gives an attacker no 
confirmation that the host is even there scanning that specific port, rather than just no service.

---

## What I Learned
- How to check firewall status and toggle it using UFW commands
- The direct, measurable impact firewall configuration has on network exposure
- Why firewall misconfiguration is a common and dangerous real-world vulnerability
- How to use Nmap comparatively — running the same scan under different 
  conditions to assess security posture changes

---

## Next Steps
- Explore configuring specific UFW rules to allow only necessary ports 
  rather than a full enable/disable approach
- Research common firewall misconfiguration incidents in real-world breaches
