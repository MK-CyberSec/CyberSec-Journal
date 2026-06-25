# Lab 005 — Access Control
**Date:** June 25, 2026
**Status:** Complete

---

## Objective
Create multiple user accounts on the Ubuntu Server VM with different 
permission levels to demonstrate practical understanding of access 
control models covered in Security+ Domain 4.

---

## Tools Used
- Ubuntu Server 26.04 LTS

---

## What I Did
Created two user accounts on the Ubuntu VM — a standard user with no 
elevated privileges and a second user added to the sudo group — then 
tested what each account could and could not do. Also created a test 
file and modified its permissions to demonstrate file-level access 
control, confirming that a restricted file became inaccessible to a 
user outside its ownership.

---

## Steps Taken
1. Created a standard user account: `sudo adduser standarduser`
2. Created a second user account: `sudo adduser adminuser`
3. Added adminuser to the sudo group: `sudo usermod -aG sudo adminuser`
4. Switched to standarduser and attempted a privileged command: 
   `sudo apt update`
5. Switched to adminuser and ran the same privileged command to confirm 
   elevated access worked correctly
6. Created a test file as serveradmin: 
   `echo "This is a test file" > testfile.txt`
7. Checked default file permissions with `ls -la`
8. Restricted the file to owner-only access: `chmod 600 testfile.txt`
9. Switched to standarduser and attempted to read the restricted file 
   with `cat`, confirming access was denied

---

## Output

**Standard user denied privileged command:**
standarduser@ubuntu-lab:~$ sudo apt update
sudo: I'm sorry standarduser. I'm afraid I can't do that
(Note: this Ubuntu installation uses a humorous "insulting sudo" 
response variant rather than the standard error message — the 
underlying denial behavior is the same.)

**Admin user successfully executing privileged command:**
adminuser@ubuntu-lab:~$ sudo apt update
[sudo: authenticate] Password:
Hit:1 http://security.ubuntu.com/ubuntu resolute-security InRelease
Hit:2 http://us.archive.ubuntu.com/ubuntu resolute InRelease
Hit:3 http://us.archive.ubuntu.com/ubuntu resolute-updates InRelease
Hit:4 http://us.archive.ubuntu.com/ubuntu resolute-backports InRelease
3 packages can be upgraded. Run 'apt list --upgradable' to see them.

**Default file permissions:**
-rw-rw-r-- 1 serveradmin serveradmin 20 Jun 25 04:57 testfile.txt

**After restricting to owner-only:**
-rw------- 1 serveradmin serveradmin 20 Jun 25 04:57 testfile.txt

**Standard user denied file access after restriction:**
standarduser@ubuntu-lab:~$ cat /home/serveradmin/testfile.txt
cat: /home/serveradmin/testfile.txt: Permission denied

---

## Issues Encountered
No issues encountered during this lab. All commands executed as 
expected on the first attempt.

---

## What This Means
This lab demonstrates two layers of access control working together. 
At the account level, sudo group membership determines whether a user 
can execute administrative commands — standarduser was correctly 
denied, while adminuser, after being added to the sudo group, was 
correctly granted access. This reflects a basic form of Role-Based 
Access Control (RBAC), where privilege is tied to group membership 
rather than the individual user.

At the file level, the permission change from `-rw-rw-r--` to 
`-rw-------` removed all access for the group and other categories, 
leaving only the file owner able to read or write it. This is 
Discretionary Access Control (DAC) — the file owner controls who can 
access their own files, and Linux enforces that boundary at the 
kernel level regardless of which user is attempting access. 
standarduser's denial when trying to read serveradmin's restricted 
file confirms that enforcement.

Together, these two mechanisms reflect the principle of least 
privilege — each account had exactly the access it was assigned, 
nothing more, and the system enforced both boundaries correctly 
without requiring any additional configuration.

---

## What I Learned
- How to create and manage Linux user accounts with `adduser`
- How to grant administrative privileges by adding a user to the sudo 
  group with `usermod -aG`
- The practical difference between a standard and privileged (sudo) 
  account when attempting the same command
- How Linux file permissions work — owner, group, and other 
  categories, each with separate read/write/execute bits
- How to modify file permissions using numeric `chmod` notation (600 
  removing all group/other access)
- How these practical mechanics connect to access control models 
  (DAC, RBAC) covered in Security+ Domain 4
- That access control enforcement in Linux is consistent and 
  immediate — no system restart or additional step was needed for 
  either the sudo restriction or the file permission restriction to 
  take effect

---

## Next Steps
- Explore Access Control Lists (ACLs) for more granular permission 
  control beyond the basic owner/group/other model
- Research how these concepts scale in enterprise environments using 
  Active Directory group policies and Group Policy Objects (GPOs)
- Investigate the principle of least privilege in the context of 
  service accounts rather than just human user accounts
