# Lab 006 — SSL Certificate Generation
**Date:** June 25, 2026
**Status:** Complete

---

## Objective
Generate a self-signed SSL certificate on the Ubuntu Server VM using 
OpenSSL to demonstrate practical understanding of PKI fundamentals 
covered in Security+ Domain 6.

---

## Tools Used
- Ubuntu Server 26.04 LTS
- OpenSSL

---

## What I Did
Generated a self-signed 4096-bit RSA certificate and private key pair 
using OpenSSL, populated the certificate's identity fields, inspected 
the resulting certificate's contents, and verified that the private 
key and certificate were a genuine matching pair using MD5 hash 
comparison.

---

## Steps Taken
1. Ran the certificate generation command:
   `openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -noenc`
2. Encountered an initial passphrase error using the unmodified command 
   (see Issues Encountered) and resolved it with the `-noenc` flag
3. Filled in the certificate's Distinguished Name fields — Country, 
   State, Locality, Organization, Common Name, and Email
4. Confirmed both `key.pem` and `cert.pem` were created using `ls -la`
5. Inspected the full certificate contents with 
   `openssl x509 -in cert.pem -text -noout`
6. Verified the private key and certificate matched using 
   `openssl x509 -noout -modulus -in cert.pem | openssl md5` and 
   `openssl rsa -noout -modulus -in key.pem | openssl md5`

---

## Output

**Files created:**
-rw-rw-r-- 1 serveradmin serveradmin 2163 Jun 25 05:05 cert.pem
-rw------- 1 serveradmin serveradmin 3268 Jun 25 05:04 key.pem

Note the differing permissions — cert.pem is group/world-readable 
(public by nature), while key.pem is restricted to owner-only 
read/write, correctly reflecting that the private key must remain 
confidential.

**Certificate details (abridged):**
Signature Algorithm: sha256WithRSAEncryption
Issuer: C=US, ST=California, L=Rancho Cucamonga, O=N/A, OU=N/A,
CN=Mustafa, emailAddress=mustafakhattak05@gmail.com
Validity
Not Before: Jun 25 05:05:05 2026 GMT
Not After : Jun 25 05:05:05 2027 GMT
Subject Public Key Info:
Public Key Algorithm: rsaEncryption
Public-Key: (4096 bit)
Exponent: 65537 (0x10001)
X509v3 Basic Constraints: critical
CA:TRUE
**Key/certificate match verification:**
MD5(stdin)= 954636d0316c3044b1beeb492eea16c0
MD5(stdin)= 954636d0316c3044b1beeb492eea16c0

Identical hashes confirm the private key and certificate are a valid, 
matching pair.

---

## Issues Encountered
Initial certificate generation attempt prompted for a PEM passphrase 
to encrypt the private key. Attempting to skip it by pressing Enter 
with no input resulted in an error — this OpenSSL version (3.x) 
requires a passphrase of 4 to 1024 characters and rejects an empty 
input outright, rather than treating it as "no passphrase" as older 
OpenSSL versions allowed. Resolved by re-running the command with the 
`-noenc` flag, which explicitly tells OpenSSL not to encrypt the 
private key, avoiding the passphrase prompt entirely. This is the 
more correct approach for OpenSSL 3.x, which expects explicit intent 
regarding key encryption rather than relying on an empty-input 
shortcut.

---

## What This Means
This lab demonstrates the foundation of how PKI (Public Key 
Infrastructure) works at the most basic level. The certificate 
contains a public key and identity information, cryptographically 
signed — in this case, self-signed rather than signed by a trusted 
Certificate Authority. The matching private key, kept separate and 
restricted to owner-only access, is what would actually be used to 
decrypt data or prove identity in a real TLS/SSL handshake.

The MD5 modulus comparison is a practical way to verify that a 
certificate and a private key genuinely belong together — a check 
that matters in real-world scenarios such as configuring a web server 
or troubleshooting an SSL pairing issue where the wrong key might 
accidentally be paired with a certificate.

The certificate generated here is self-signed and marked CA:TRUE by 
default under the `-x509` flag — meaning it could technically sign 
other certificates, unlike a typical production server certificate 
issued by a trusted CA, which is correctly restricted from doing so. 
This distinction matters in real PKI deployments, where over-permissive 
certificate constraints are a genuine security risk.

---

## What I Learned
- How to generate a self-signed RSA certificate and private key pair 
  using OpenSSL
- The difference between a certificate (public, shareable) and a 
  private key (confidential, restricted) reflected directly in their 
  file permissions
- How to inspect a certificate's full contents, including validity 
  period, subject information, and public key details
- How to verify that a private key and certificate are a genuine 
  matching pair using MD5 modulus comparison
- That OpenSSL 3.x requires explicit handling of key encryption — an 
  empty passphrase is rejected rather than silently accepted as in 
  older versions
- The distinction between a self-signed CA-capable certificate and a 
  standard end-entity certificate, and why production certificates are 
  typically restricted from acting as a CA

---

## Next Steps
- Generate a Certificate Signing Request (CSR) and explore the 
  difference between self-signing and submitting to an actual 
  Certificate Authority
- Investigate configuring a web server (e.g. Apache or Nginx) to 
  actually use this certificate for HTTPS
- Research the role of intermediate certificates and certificate 
  chains in real-world TLS deployments
