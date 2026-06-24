# Lab Notes — VM Access Recovery
**Date:** June 2026
**Status:** Resolved

---

## Context
Before starting Lab 002, I lost track of the login credentials set during initial VM buildout for both the Kali Linux and Ubuntu Server machines in my home lab. Since both VMs were early-stage builds with no production data or completed work on them, I had latitude to choose between a proper credential reset and a full rebuild — and used a different approach for each, depending on what made sense for that machine.

---

## Machine 1: Kali Linux — Password Reset via GRUB

Kali was my primary attack-box VM with some tool configuration already in place, so a rebuild would have cost more time than a reset. I recovered access directly:

1. Rebooted the VM and interrupted boot to reach the GRUB menu
2. Selected the default boot entry and pressed `e` to edit it
3. Located the line beginning with `linux` and changed `ro quiet splash` to `rw init=/bin/bash`
4. Booted the edited entry (`Ctrl+X`), landing at a root shell with no login required
5. Ran `passwd <username>` to set a new password
6. Reloaded init (`exec /sbin/init`) and confirmed login with the new credentials

---

## Machine 2: Ubuntu Server — Rebuild

Ubuntu Server was the target machine for scan-based labs, with no meaningful configuration invested in it yet. Rather than run a parallel recovery process, I deleted the VM and rebuilt it from a fresh ISO. This was faster than recovery and gave me a clean baseline for the scanning work in Lab 002.

---

## Takeaways
- **Reset vs. rebuild is a judgment call, not a default.** Reset made sense for Kali because of the configuration already invested in it; rebuild made sense for Ubuntu because there was nothing to preserve. Knowing when *not* to spend time on recovery is as much a skill as knowing how to do the recovery.
- **Root cause:** credentials weren't recorded anywhere outside my own memory during initial setup.
- **Fix going forward:** credentials for any new lab VM get logged immediately — in a password manager — before moving on to the next step of setup.

---

## Next Steps
- Apply this credential-logging habit to all future VM builds
- Proceed to Lab 002 (Nmap Service Scan) using the rebuilt Ubuntu Server VM
