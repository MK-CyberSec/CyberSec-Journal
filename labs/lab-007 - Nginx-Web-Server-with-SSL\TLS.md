# Lab 007 — Nginx Web Server with SSL/TLS
**Date:** June 25, 2026
**Status:** Complete

---

## Objective
Configure Nginx as a web server on the Ubuntu Server VM and enable 
HTTPS using the self-signed SSL certificate generated in Lab 006, 
demonstrating practical understanding of how TLS/SSL certificates 
are deployed on real web servers.

---

## Tools Used
- Ubuntu Server 26.04 LTS
- Nginx
- OpenSSL (certificate from Lab 006)

---

## What I Did
Installed Nginx on the Ubuntu VM, confirmed basic HTTP access, then 
configured the server to serve content over HTTPS using the 
self-signed certificate and private key generated in Lab 006. Verified 
the configuration was correct by accessing the server from a host 
machine browser over HTTPS and confirming the certificate details 
matched those set during generation.

---

## Steps Taken
1. Installed Nginx: `sudo apt install nginx -y`
2. Confirmed Nginx was running: `sudo systemctl status nginx`
3. Verified HTTP access from host browser at http://192.168.56.106
4. Created SSL directory and moved certificate files into place:
   `sudo mkdir -p /etc/nginx/ssl`
   `sudo mv ~/cert.pem /etc/nginx/ssl/`
   `sudo mv ~/key.pem /etc/nginx/ssl/`
5. Edited Nginx site configuration to add an HTTPS server block
6. Encountered configuration errors due to leftover orphaned 
   directives in the default config file (see Issues Encountered)
7. Overwrote the config file cleanly using `tee` to guarantee no 
   residual content remained
8. Tested configuration: `sudo nginx -t`
9. Opened port 443: `sudo ufw allow 443/tcp`
10. Restarted Nginx: `sudo systemctl restart nginx`
11. Accessed https://192.168.56.106 from host browser, clicked 
    through self-signed certificate warning, and confirmed Nginx 
    welcome page served over HTTPS
12. Verified certificate details in browser matched Lab 006 
    certificate exactly

---

## Output

**Nginx configuration test:**
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

**Final Nginx configuration (/etc/nginx/sites-available/default):**
```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}

server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;

    server_name ubuntu-lab;

    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;

    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Browser certificate details confirmed:**
Common Name (CN): Mustafa
Organization (O): N/A
Organizational Unit (OU): N/A
Issued On: Wednesday, June 24, 2026 at 10:05:05 PM
Expires On: Thursday, June 24, 2027 at 10:05:05 PM

Browser error received and expected:
net::ERR_CERT_AUTHORITY_INVALID

This confirms the browser successfully received and evaluated the 
certificate, rejecting it only because it is self-signed rather than 
issued by a trusted Certificate Authority — the correct behavior for 
a self-signed certificate in a lab environment.

---

## Issues Encountered

**1. Nginx configuration syntax error after initial edit**
After adding the HTTPS server block via nano, Nginx's configuration 
test returned: "server_name" directive is not allowed here in /etc/nginx/sites-enabled/default:56
Diagnosed by inspecting the full file contents — the new HTTPS block 
had been inserted correctly but the original HTTP server block had 
lost its opening `server {` and `listen 80;` lines in the process, 
leaving the remaining directives as orphaned content floating outside 
any server block. Nginx interpreted these as top-level directives, 
which is invalid.

**2. Repeated nano edits failed to fully remove residual content**
Multiple attempts to manually clean the file via nano resulted in 
leftover content persisting below the new blocks each time, continuing 
to cause the same syntax error. Resolved by bypassing nano entirely 
and using `tee` with a heredoc to overwrite the file completely in 
one command, guaranteeing only the intended configuration remained: sudo tee /etc/nginx/sites-available/default << 'EOF'
[config content]
EOF
This approach eliminates the risk of residual content surviving an 
edit session and is more reliable than manual deletion for wholesale 
file replacement.

---

## What This Means
This lab connects Lab 006's theoretical certificate generation to 
a real-world deployment scenario. A certificate sitting as a .pem 
file on disk does nothing on its own — it only becomes meaningful 
when a service is configured to actually use it. Nginx acting as 
the TLS termination point means all traffic between the client 
browser and the server is encrypted in transit, even though the 
underlying content (the Nginx welcome page) is completely static.

The browser's ERR_CERT_AUTHORITY_INVALID warning is a practical 
demonstration of the chain of trust concept in PKI — browsers 
maintain a built-in list of trusted Certificate Authorities, and 
any certificate not signed by one of those authorities is flagged 
regardless of whether the certificate itself is technically valid. 
This is why production websites use certificates from recognised 
CAs like Let's Encrypt rather than self-signed ones, even though 
the encryption strength is identical.

---

## What I Learned
- How to install and configure Nginx as a web server on Ubuntu
- How to structure an Nginx configuration file with multiple server 
  blocks handling different ports and protocols
- How to configure Nginx to use an existing SSL certificate and 
  private key for HTTPS
- Why browsers display certificate warnings for self-signed certs 
  and what that warning actually means technically
- The practical difference between a certificate being 
  cryptographically valid and being trusted by a browser — 
  validity and trust are separate concepts in PKI
- How to use `tee` with a heredoc as a reliable method for 
  completely overwriting a file's contents from the terminal, 
  bypassing editor-related issues
- How to test Nginx configuration syntax before restarting the 
  service, preventing unnecessary downtime from invalid configs

---

## Next Steps
- Obtain a free trusted certificate from Let's Encrypt using 
  Certbot and replace the self-signed cert to eliminate the 
  browser warning
- Configure HTTP to HTTPS redirect so port 80 traffic is 
  automatically redirected to port 443
- Research SSL/TLS cipher suite configuration to understand 
  how to harden HTTPS beyond the default Nginx settings
