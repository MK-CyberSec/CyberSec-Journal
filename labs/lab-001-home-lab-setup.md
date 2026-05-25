# Lab 001 — Home Lab Setup
**Date:** May 25, 2026  
**Status:** Complete

---

## Objective
Build a virtualised home lab environment consisting of an attacker 
machine (Kali Linux) and a target machine (Ubuntu Server), networked 
together privately using VirtualBox.

---

## Tools Used
- VirtualBox 7.2.8
- Kali Linux 2026.1 (pre-built VirtualBox image)
- Ubuntu Server 26.04 LTS

---

## What I Built
Two virtual machines running inside VirtualBox on my personal machine:

**Kali Linux VM** — attacker machine. Pre-loaded with penetration 
testing and security tools. This is the machine I will use to run 
scans, test exploits, and practice offensive security techniques.

**Ubuntu Server VM** — target machine. A basic Linux server acting 
as the victim in lab exercises. Completely isolated from my real 
network.

Both machines are connected via a Host-Only Adapter network in 
VirtualBox — meaning they can talk to each other but neither can 
reach the real internet. This keeps all testing contained and safe.

---

## Steps Taken
1. Downloaded and installed VirtualBox 7.2.8 from virtualbox.org
2. Downloaded Kali Linux pre-built .ova image from kali.org/get-kali
3. Attempted to import Kali via File → Import Appliance — function failed. Resolved by navigating to the extracted Kali folder and launching directly from the .vbox Machine Definition file.
4. Downloaded Ubuntu Server 26.04 LTS ISO from ubuntu.com
5. Created new VM in VirtualBox, skipped unattended installation
6. Set base memory to 5000MB
7. Completed manual Ubuntu Server installation
8. Set both VMs to Host-Only Adapter in network settings
9. Booted both machines
10. Found Ubuntu IP address using: ip a
11. From Kali terminal ran: ping 192.168.56.105
12. Received successful responses confirming lab is live

---

## Output
Ping response from Kali to Ubuntu:
```
PING 192.168.56.105 (192.168.56.105) 56(84) bytes of data.
64 bytes from 192.168.56.105: icmp_seq=1 ttl=64 time=0.774 ms
64 bytes from 192.168.56.105: icmp_seq=2 ttl=64 time=0.731 ms
64 bytes from 192.168.56.105: icmp_seq=3 ttl=64 time=0.694 ms
```
Ubuntu IP: 192.168.56.105
Response times: 0.774ms, 0.731ms, 0.694ms — clean and consistent, no packet loss

---

## Issues Encountered
- VirtualBox Import Appliance function failed to import Kali .ova 
  successfully. Resolved by navigating to the extracted Kali folder 
  and launching directly from the .vbox Machine Definition file. 
  Same result achieved through an alternative method.
- Ubuntu installation froze on first attempt due to unattended 
  installation being enabled. Deleted VM and rebuilt with unattended 
  installation unchecked. Resolved immediately.
- Ubuntu VM unable to ping external IPs or resolve domain names. 
  Identified as expected behaviour of Host-Only networking — VMs 
  are intentionally isolated from the internet. Not a lab issue.

---

## What This Means
A successful ping confirms two things:
1. Both VMs are running and reachable on the same private network
2. The lab environment is functional and ready for security exercises

The ICMP protocol used by ping is one of the most basic forms of 
network communication. Successfully reaching the target machine 
means I can now run more advanced tools like Nmap against it in 
future labs.

---

## What I Learned
- How VirtualBox creates isolated network environments using 
  Host-Only Adapters
- The difference between a live environment and a virtualised one 
  and why isolation matters for safe security practice
- Basic Linux networking — using ip a to find an IP address and 
  ping to test connectivity
- Host-Only networking intentionally blocks external internet 
  access — this is by design, not a misconfiguration
- Unattended installation in VirtualBox can cause issues with 
  newer Ubuntu versions — manual installation is more reliable

---

## Next Steps
- Run first Nmap scan against Ubuntu VM (Lab 002)
- Begin practising port enumeration and service detection
- Continue TryHackMe Pre-Security path
