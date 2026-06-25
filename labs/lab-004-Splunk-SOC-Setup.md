# Lab 004 — Splunk SOC Setup
**Date:** June 24-25, 2026
**Status:** Complete

---

## Objective
Install Splunk Free on the Ubuntu Server VM, configure it to ingest 
system logs, and build a basic alert to simulate a functioning 
mini Security Operations Center (SOC) monitoring environment.

---

## Tools Used
- Ubuntu Server 26.04 LTS
- Kali Linux 2026.1
- Splunk Free (Enterprise trial / free tier)

---

## What I Did
Installed Splunk on the Ubuntu target machine and configured it to 
ingest local system logs, starting with syslog before discovering 
that SSH authentication events are logged separately to auth.log. 
Added auth.log as a second data input, generated test activity by 
intentionally triggering failed SSH login attempts, and confirmed 
those events were indexed correctly. Built and tested a real-time 
alert that successfully detected a failed login attempt, simulating 
the foundational detection workflow of a SOC analyst.

---

## Steps Taken
1. Downloaded Splunk Free from splunk.com onto the Ubuntu VM (via Kali, 
   after resolving multiple download issues — see Issues Encountered)
2. Installed Splunk using the provided .deb installation package
3. Started the Splunk service and accessed the web interface
4. Configured an initial data input monitoring /var/log/syslog
5. Ran a baseline search (`index=*`) confirming 5,772 historical events 
   had already been indexed
6. Generated test activity by intentionally triggering failed SSH login 
   attempts against the Ubuntu VM
7. Discovered failed login attempts were not appearing in syslog and 
   added a second data input monitoring /var/log/auth.log
8. Confirmed failed login events appeared correctly under the new input
9. Created an alert named "Failed SSH Login Attempt," triggering on 
   any search result for `index=* "Failed password"`
10. Tested the alert by generating another failed login attempt and 
    confirming it triggered and logged correctly under Triggered Alerts

---

## Output

**Initial data input:**
- Monitor configured: /var/log/syslog

**Baseline ingestion confirmation:**
Search query: `index=*`
Result: Completed — 5,772 events (6/24/26 3:00:00.000 AM to 6/25/26 
3:35:15.000 AM), confirming Splunk successfully backfilled and indexed 
existing system log history immediately upon configuring the monitor.

**Secondary data input (added after troubleshooting):**
- Monitor configured: /var/log/auth.log
- Reason: SSH authentication failures are logged here, not in syslog

**Alert configuration:**
- Name: Failed SSH Login Attempt
- Search basis: `index=* "Failed password"`
- Trigger condition: Number of results greater than 0
- Trigger mode: Real-time
- Action: Add to Triggered Alerts

**Alert test — triggered successfully:**
6/25/26 3:43:42.694 AM
2026-06-25T03:43:42.694126+00:00 ubuntu-lab sshd-session[8165]:
Failed password for serveradmin from 192.168.56.101 port 48922 ssh2
host = ubuntu-lab
source = /var/log/auth.log
sourcetype = AuthLog

The alert correctly identified a failed SSH authentication attempt in 
real time, capturing the exact timestamp, source IP address, and 
target username — demonstrating the same fundamental detection 
mechanism used in real-world SOC environments to flag potential 
brute-force or unauthorized access attempts.

---

## Issues Encountered
This installation involved an unusually long chain of distinct, layered 
issues, each one masking the next until isolated individually. 
Documented in the order encountered:

**1. No internet access on Ubuntu Server VM**
The Ubuntu VM was deliberately configured with only a Host-Only Adapter, 
which by design isolates it from the internet. Splunk's installer 
needed to be downloaded, requiring temporary internet access. Resolved 
by adding a second network adapter (NAT) to the VM, leaving the 
original Host-Only adapter untouched so the private lab network 
remained unaffected.

**2. wget returning 404 Not Found on a valid URL**
Initial wget download attempts against Splunk's download server 
returned 404 Not Found errors despite the URL being valid and confirmed 
working in a browser. Diagnosed by comparing browser behavior to 
wget's default request — hypothesized the server was filtering based 
on the User-Agent header rather than rejecting the URL itself. Resolved 
by adding a --user-agent flag to wget, spoofing a standard browser 
identification string. This confirmed the server was using User-Agent 
filtering, likely as a basic anti-automation/anti-scraping measure, 
rather than the link itself being broken.

**3. Downloaded file was 0 bytes despite wget reporting success**
After resolving the User-Agent filtering issue, wget reported a 
successful download, but the resulting file was 0 bytes. Diagnosed 
using the `file` and `ls -lh` commands, confirming the file was empty 
despite wget reporting success. Concluded that Splunk's download 
infrastructure likely requires session/cookie handling or redirect 
behavior that wget could not replicate even with a spoofed User-Agent. 
Resolved by downloading the file via browser on the host machine and 
transferring it to the Ubuntu VM using scp over the existing SSH 
connection.

**4. Kali Linux also had no internet access**
The same Host-Only isolation applied to Kali, preventing it from being 
used as an alternative download source either. Resolved by adding a 
second NAT adapter to Kali as well, which also provides ongoing 
internet access for future lab work requiring tool updates or 
downloads.

**5. Disk full during installation — "No space left on device"**
After successfully transferring the installer to Ubuntu via scp, dpkg 
failed mid-install due to insufficient disk space. Investigation with 
`df -h` revealed the root filesystem showed only 12GB available despite 
the VM being configured with a 20GB virtual disk. Further investigation 
with `vgs` revealed Ubuntu's LVM-based installer had only allocated 
roughly half the disk to the active logical volume, leaving the 
remainder unallocated and inaccessible. Resolved using `lvextend` to 
claim the unallocated space and `resize2fs` to extend the filesystem to 
use it.

**6. Splunk refused to run as root without explicit flag**
Splunk's startup process detected it was being run as the root user and 
halted with a deprecation warning, requiring an explicit --run-as-root 
flag to proceed. Resolved by including the flag in the start command.

**7. No admin account created on first start**
Using --no-prompt --answer-yes flags to bypass interactive setup also 
skipped admin account creation entirely, since no password was supplied 
through that path. Resolved by creating a user-seed.conf file under 
Splunk's local configuration directory to pre-seed an admin username 
and password on next startup.

**8. Web interface never became available despite splunkd running**
After multiple restart attempts, splunkd reported as running, but the 
web interface on port 8000 never became reachable, both through direct 
browser access and confirmed absent via `ss -tlnp`. This took the most 
extensive troubleshooting of the night and several stacked causes were 
ultimately responsible:

   - **Duplicate splunkd processes**: an earlier stop command had not 
     actually terminated the original process, and a second start 
     attempt was layered on top of the still-running first instance, 
     creating a conflict. Resolved by force-killing all Splunk-related 
     processes directly with pkill -9 and confirming a clean process 
     table before restarting.

   - **Insufficient VM memory**: after rebuilding the Ubuntu VM earlier 
     in the project, the memory allocation had silently reset to a 
     default value far below the originally configured 5000MB. Checked 
     with `free -h`, which showed only 1.6GB total memory available — 
     well below Splunk's recommended minimum. This left the system 
     under memory pressure, with the web component likely failing to 
     fully initialize as a result. Resolved by manually re-checking and 
     correcting the VM's memory allocation in VirtualBox settings (and 
     proactively increasing Kali's allocation as well, given 32GB of 
     total host RAM was available to allocate from).

**9. "No users exist" error on login despite a configured user-seed.conf**
After finally getting the web interface to bind successfully, login 
failed with no admin account existing. Investigation revealed the 
seeded password did not meet Splunk's minimum 8-character requirement, 
which caused the seed file to silently fail validation without any 
visible error during startup. Resolved by updating the password to 
meet the minimum length requirement and forcing a genuine cold restart 
(via pkill, since stop was again not reliably terminating the running 
instance) to ensure the corrected seed file was actually read.

**10. Alert not triggering despite generating failed login attempts**
After configuring the alert based on a search against the syslog data 
input, intentionally generated failed SSH login attempts did not 
appear in search results. Diagnosed by checking /var/log/auth.log 
directly on Ubuntu, which confirmed SSH authentication failures are 
logged there rather than to syslog by default. Resolved by adding a 
second Splunk data input monitoring auth.log specifically, after which 
the failed login events appeared correctly and the alert triggered as 
expected.

---

## What This Means
A SIEM (Security Information and Event Management) tool like Splunk 
centralises log data from systems and applications, allowing security 
analysts to detect suspicious activity that would otherwise be buried 
across dozens of individual log files. The alert configured in this 
lab demonstrates the foundational SOC workflow — defining a condition 
worth flagging, then automating detection of that condition rather 
than manually reviewing logs. The fact that the alert initially failed 
to trigger due to a log source mismatch also reflects a real-world 
SOC challenge: a monitoring system is only as effective as its 
visibility into the correct data sources, and assuming full coverage 
without verifying it can create dangerous blind spots.

---

## What I Learned
- How to install and configure Splunk on a Linux system
- How log ingestion works — pointing Splunk at specific log files or 
  sources
- How to build a basic correlation-based alert
- Why centralised logging is foundational to SOC operations
- That different categories of system events are often routed to 
  different log files by default (e.g. authentication events to 
  auth.log rather than syslog), and that data source selection in a 
  SIEM must account for this rather than assuming a single log file 
  captures everything
- How to add and configure a second network adapter in VirtualBox to 
  grant internet access to an otherwise isolated VM without disturbing 
  an existing Host-Only lab network
- How HTTP User-Agent headers can be used by servers to filter or block 
  automated tools like wget and curl
- How to diagnose a server-side filtering issue by comparing successful 
  vs failed request behavior rather than assuming a broken link
- How to use wget's --user-agent flag to modify request headers
- That a "successful" download reported by wget does not guarantee a 
  valid file — always verify file size and type before trusting a 
  download completed correctly
- How LVM (Logical Volume Management) can leave disk space unallocated 
  after installation, and how to identify and reclaim that space using 
  vgs, lvextend, and resize2fs
- That VirtualBox VM memory allocation can silently reset to default 
  values after a VM is deleted and rebuilt, and that this should always 
  be manually reconfirmed rather than assumed
- How to properly verify a process is actually terminated using 
  ps aux before attempting to restart a service, rather than trusting 
  a stop command's exit silently
- How to forcibly terminate stuck or unresponsive processes using 
  pkill -9 when a service's own stop command fails to do so
- That configuration files like user-seed.conf can fail validation 
  silently (in this case due to password length) without producing any 
  visible error, reinforcing the importance of checking actual results 
  rather than assuming a config was applied correctly
- The overall value of systematic elimination when troubleshooting a 
  single symptom with multiple possible causes — checking processes, 
  ports, logs, memory, and configuration individually rather than 
  guessing at a single fix

---

## Next Steps
- Configure email or webhook-based alert actions instead of relying 
  solely on the Triggered Alerts dashboard
- Explore ingesting additional log sources beyond syslog and auth.log
- Research more advanced Splunk search query syntax (SPL)
- Investigate Splunk's built-in dashboards for visualising ingested 
  data rather than relying solely on raw search queries
