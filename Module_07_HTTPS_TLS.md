# Module 7 — HTTPS + TLS

> **HIGH PRIORITY MODULE**

---

## 1. HTTP vs HTTPS `MUST KNOW`

| Feature | HTTP | HTTPS |
|---------|------|-------|
| Full name | HyperText Transfer Protocol | HyperText Transfer Protocol **Secure** |
| Port | 80 | 443 |
| Encryption | None — plaintext | TLS encryption |
| Security | Anyone on the network can read/modify data | Encrypted, authenticated, integrity-checked |
| URL scheme | `http://` | `https://` |

**HTTPS = HTTP + TLS.** The HTTP protocol is identical — TLS is a layer between HTTP and TCP that encrypts everything.

---

## 2. Why HTTPS Exists `MUST KNOW`

HTTPS provides three security guarantees:

| Goal | What It Means | Without It |
|------|--------------|------------|
| **Confidentiality** | Data is encrypted — can't be read by eavesdroppers | Anyone on the network (ISP, Wi-Fi attacker) can read your passwords, emails, messages |
| **Integrity** | Data can't be modified in transit without detection | Attacker can inject malware, ads, or alter page content |
| **Authentication** | You're talking to the real server, not an impersonator | Attacker can impersonate google.com and steal your credentials |

---

## 3. Encryption Types `MUST KNOW`

### Symmetric Encryption
- **Same key** for encryption and decryption.
- Both parties must share the secret key.
- **Fast** — efficient for encrypting large amounts of data.
- Algorithms: AES, ChaCha20.
- **Problem:** How do you share the key securely over an insecure channel?

```
Plaintext → [Encrypt with Key K] → Ciphertext → [Decrypt with Key K] → Plaintext
```

### Asymmetric Encryption (Public Key Cryptography)
- **Two keys:** Public key (anyone can have) and Private key (only owner has).
- Data encrypted with the **public key** can only be decrypted with the **private key**.
- **Slow** compared to symmetric.
- Algorithms: RSA, ECDHE (Elliptic Curve Diffie-Hellman Ephemeral).
- Also used for **digital signatures** — sign with private key, verify with public key.

```
Plaintext → [Encrypt with Public Key] → Ciphertext → [Decrypt with Private Key] → Plaintext
```

### Why Both? (Hybrid Encryption) `MUST KNOW`
- Asymmetric encryption is too **slow** for bulk data.
- Symmetric encryption has the **key distribution** problem.
- **Solution:** Use asymmetric encryption to securely exchange a symmetric key, then use symmetric encryption for the actual data.

```
1. TLS handshake: Asymmetric crypto → agree on a shared secret → derive symmetric "session keys"
2. Data transfer: Symmetric crypto with session keys (fast)
```

**Interview answer:** "HTTPS uses hybrid encryption. During the TLS handshake, asymmetric cryptography (like Diffie-Hellman) securely establishes a shared secret between client and server. From this, both sides derive symmetric session keys. All subsequent data is encrypted with these symmetric keys because symmetric encryption is much faster. This gives us the best of both — secure key exchange from asymmetric, fast bulk encryption from symmetric."

---

## 4. Digital Certificates `MUST KNOW`

### What a Certificate Contains
- **Domain name** (e.g., `google.com`)
- **Public key** of the server
- **Issuer** — which Certificate Authority (CA) issued it
- **Validity period** (not before / not after dates)
- **Digital signature** — the CA's signature proving the certificate is authentic
- **Subject Alternative Names (SAN)** — additional domains covered

### Why Certificates Exist
Without certificates, you have no way to verify that the public key you received actually belongs to `google.com`. An attacker could give you their own public key and intercept everything. The certificate, signed by a trusted CA, proves the public key belongs to the real server.

---

## 5. Certificate Authority (CA) `MUST KNOW`

A **trusted third party** that issues and signs digital certificates.

### How It Works
1. Server owner generates a key pair (public + private key).
2. Server owner sends a **Certificate Signing Request (CSR)** to a CA.
3. CA **verifies domain ownership** (via DNS record, email, or HTTP challenge).
4. CA signs the certificate with the **CA's private key**.
5. Server installs the signed certificate.

### Chain of Trust
```
Root CA (self-signed, trusted by OS/browser)
    ↓ signs
Intermediate CA
    ↓ signs
Server Certificate (for google.com)
```

- **Root CAs** are pre-installed in your OS/browser (trust store).
- Root CAs sign Intermediate CAs.
- Intermediate CAs sign server certificates.
- Verification: browser traces the chain from server cert → intermediate → root. If the root is trusted → certificate is valid.

### Certificate Verification (what the browser checks)
1. **Signature valid?** — Verify the CA's digital signature using the CA's public key.
2. **Domain matches?** — Certificate's domain matches the website you're visiting.
3. **Not expired?** — Current date is within the validity period.
4. **Not revoked?** — Check CRL (Certificate Revocation List) or OCSP.
5. **Trusted CA?** — The signing CA traces back to a trusted root CA.

---

## 6. TLS Handshake `MUST KNOW`

### Simplified TLS 1.2 Handshake

```
Client                                          Server
  |                                               |
  |  1. Client Hello                              |
  |  (TLS versions, cipher suites, random_c) ──>  |
  |                                               |
  |  2. Server Hello                              |
  |  <── (chosen version, cipher, random_s)       |
  |                                               |
  |  3. Server Certificate                        |
  |  <── (certificate with public key)            |
  |                                               |
  |  4. Server Key Exchange + Server Hello Done   |
  |  <── (DH parameters)                          |
  |                                               |
  |  [Client verifies certificate]                |
  |                                               |
  |  5. Client Key Exchange                       |
  |  (DH parameters) ──>                          |
  |                                               |
  |  [Both derive shared secret → session keys]   |
  |                                               |
  |  6. Change Cipher Spec + Finished             |
  |  ──> (encrypted "Finished" message)           |
  |                                               |
  |  7. Change Cipher Spec + Finished             |
  |  <── (encrypted "Finished" message)           |
  |                                               |
  |  ═══ Encrypted communication begins ═══      |
```

**Step by step:**

1. **Client Hello:** Client sends supported TLS versions, cipher suites, and a client random number.
2. **Server Hello:** Server picks TLS version and cipher suite, sends its random number.
3. **Certificate:** Server sends its digital certificate (containing its public key).
4. **Key Exchange:** Server sends Diffie-Hellman parameters.
5. **Client Key Exchange:** Client sends its DH parameters. Both sides now independently compute the **same shared secret** (without ever transmitting it).
6. **Session Keys:** Both sides derive symmetric session keys from: shared secret + client random + server random.
7. **Finished:** Both send encrypted "Finished" messages to verify the handshake succeeded.

**TLS 1.2 handshake = 2 RTTs** (before any HTTP data).

### TLS 1.3 Improvements `GOOD TO KNOW`

| Feature | TLS 1.2 | TLS 1.3 |
|---------|---------|---------|
| Handshake | 2 RTTs | **1 RTT** |
| Resumption | Session tickets | **0-RTT** for returning clients |
| Cipher suites | Many (including insecure ones) | Only strong, modern suites |
| Forward secrecy | Optional | **Mandatory** (always Diffie-Hellman) |
| Key exchange | RSA or DH | **DH only** (no RSA key exchange) |

**Forward secrecy:** Even if the server's private key is compromised later, past session keys can't be recovered (because DH generates ephemeral keys per session).

---

## 7. Complete HTTPS Connection Flow `MUST KNOW`

**What happens when you connect to `https://example.com`:**

```
Step 1: DNS Lookup                              [Application Layer, UDP]
   Browser → DNS resolver → Root → TLD → Auth
   Result: example.com = 93.184.216.34

Step 2: TCP 3-Way Handshake                     [Transport Layer]
   Client → SYN → Server
   Client ← SYN-ACK ← Server
   Client → ACK → Server
   (1 RTT)

Step 3: TLS Handshake                           [Between Transport & Application]
   Client Hello → Server Hello → Certificate →
   Key Exchange → Session Keys → Finished
   (1-2 RTTs depending on TLS version)

Step 4: HTTP Request (Encrypted)                [Application Layer]
   GET / HTTP/1.1
   Host: example.com
   (encrypted by TLS, segmented by TCP, packetized by IP)

Step 5: HTTP Response (Encrypted)               [Application Layer]
   HTTP/1.1 200 OK
   Content-Type: text/html
   (encrypted, travels back through the stack)

Step 6: Browser Renders
   Decrypts → Parses HTML → Requests CSS/JS/images → Renders page
```

**Where each protocol fits:**

```
Application:  HTTP (request/response)
Security:     TLS  (encryption)
Transport:    TCP  (reliable delivery)
Network:      IP   (routing)
Link:         Ethernet (local delivery)

DNS happens first (separate UDP query), then TCP, then TLS, then HTTP.
```

---

## 8. MITM Attack `GOOD TO KNOW`

**Man-in-the-Middle:** An attacker sits between client and server, intercepting and potentially modifying communication.

**Without HTTPS:**
- Attacker can read all data (passwords, messages, credit cards).
- Attacker can modify data (inject malware, change page content).

**With HTTPS:**
- Data is encrypted → attacker can't read it.
- Integrity checks → attacker can't modify it.
- Certificate verification → attacker can't impersonate the server (they can't get a valid certificate for `google.com`).

**Exception:** If an attacker installs a **rogue root CA** on your machine (corporate proxies do this), they CAN issue fake certificates and intercept HTTPS traffic. This is how corporate network inspection works.

---

## Interview Questions + Answers

---

**Q1: HTTP vs HTTPS?**

**Ideal Answer:**
"HTTPS is HTTP over TLS. HTTP sends data in plaintext — anyone on the network can read it. HTTPS encrypts all communication using TLS, providing confidentiality (data can't be read), integrity (data can't be modified), and authentication (you're talking to the real server). HTTPS uses port 443; HTTP uses port 80."

---

**Q2: Why does HTTPS exist? What does it protect against?**

**Ideal Answer:**
"HTTPS protects against eavesdropping, data tampering, and impersonation. Without it, anyone on the network path (ISP, Wi-Fi attacker, compromised router) can read passwords and data, inject malicious content, or impersonate websites. HTTPS ensures confidentiality through encryption, integrity through MACs, and authentication through certificates."

---

**Q3: What is symmetric vs asymmetric encryption?**

**Ideal Answer:**
"Symmetric uses the same key for encryption and decryption — fast but requires both parties to have the shared key. Asymmetric uses a public/private key pair — anyone can encrypt with the public key, only the private key holder can decrypt. Asymmetric is slower but solves the key distribution problem. HTTPS uses both: asymmetric for key exchange during the TLS handshake, then symmetric for fast bulk data encryption."

---

**Q4: Why use both symmetric and asymmetric encryption?**

**Ideal Answer:**
"Asymmetric encryption solves the key distribution problem but is too slow for bulk data. Symmetric encryption is fast but you need a way to securely share the key. TLS uses asymmetric cryptography (Diffie-Hellman) during the handshake to establish a shared secret, then derives symmetric session keys for the actual data transfer. This combines the security of asymmetric with the speed of symmetric."

---

**Q5: What is a digital certificate?**

**Ideal Answer:**
"A digital certificate binds a server's identity (domain name) to its public key, signed by a trusted Certificate Authority. It contains the domain, public key, issuer, validity period, and the CA's digital signature. When you connect to a server, it presents its certificate. Your browser verifies the CA's signature, checks the domain matches, and confirms it's not expired. This proves you're talking to the real server and not an impersonator."

---

**Q6: What does a Certificate Authority do?**

**Ideal Answer:**
"A CA is a trusted third party that verifies domain ownership and issues signed certificates. It verifies that the certificate requestor actually owns the domain, then signs the certificate with the CA's private key. Browsers trust a set of root CAs — when a server presents a certificate signed by a trusted CA (or its chain traces back to one), the browser trusts it. This creates a chain of trust from root CA → intermediate CA → server certificate."

---

**Q7: How does the TLS handshake work?**

**Ideal Answer:**
"The client sends a Client Hello with supported TLS versions and cipher suites. The server responds with Server Hello, picking the cipher suite, and sends its certificate. Both sides perform a key exchange (typically Diffie-Hellman) to establish a shared secret without transmitting it. From this shared secret plus random numbers from both sides, they derive symmetric session keys. Both send encrypted 'Finished' messages to verify success. All subsequent HTTP data is encrypted with these session keys."

---

**Q8: What are session keys?**

**Ideal Answer:**
"Session keys are symmetric encryption keys derived during the TLS handshake, unique to each connection. They're derived from the shared secret (established via Diffie-Hellman) and random numbers from both sides. Session keys are used to encrypt all data for that session and are discarded when the connection closes. Using per-session keys means compromising one session doesn't compromise others."

---

**Q9: What happens when you connect to https://example.com?**

**Ideal Answer:**
"First, DNS resolves `example.com` to an IP address. Then a TCP 3-way handshake establishes a connection (1 RTT). Then TLS handshake: client and server exchange hellos, server sends its certificate, they perform key exchange to derive session keys (1-2 RTTs). Now the connection is encrypted. The browser sends an HTTP GET request (encrypted by TLS), the server responds with the page content (encrypted). The browser decrypts and renders the page."

---

**Q10: Can someone intercept HTTPS traffic?**

**Ideal Answer:**
"Under normal circumstances, no. The data is encrypted so an eavesdropper can't read it, and certificate verification prevents impersonation. However, if an attacker can install a trusted root CA on your device (like corporate proxies do), they can issue fake certificates and perform a man-in-the-middle attack. Also, if the server's private key is compromised and the connection doesn't use forward secrecy, past traffic could be decrypted."

---

**Q11: What is forward secrecy?**

**Ideal Answer:**
"Forward secrecy (or perfect forward secrecy) ensures that even if a server's private key is compromised in the future, past session keys can't be recovered. This is achieved using ephemeral Diffie-Hellman key exchange — each session generates temporary key pairs that are discarded after use. TLS 1.3 mandates forward secrecy; TLS 1.2 supports it optionally."

---

**Q12: What is the difference between TLS 1.2 and TLS 1.3?**

**Ideal Answer:**
"TLS 1.3 is faster and more secure. The handshake takes 1 RTT instead of 2, with 0-RTT resumption for returning clients. TLS 1.3 removes insecure cipher suites and mandates forward secrecy (always uses Diffie-Hellman, no RSA key exchange). It's simpler with fewer options, reducing the attack surface."

---

### Common Mistakes

| Mistake | Correction |
|---------|------------|
| "HTTPS encrypts the URL / domain name" | The **domain** is visible in the TLS Client Hello (SNI) and in DNS queries. HTTPS encrypts the HTTP request/response (path, headers, body), but not the domain name in the initial handshake. |
| "HTTPS encrypts DNS queries" | DNS is a **separate protocol**. Traditional DNS is plaintext (UDP port 53). DNS over HTTPS (DoH) encrypts DNS, but that's separate from HTTPS itself. |
| "Asymmetric encryption is used for all data" | Asymmetric is only used for **key exchange** during the handshake. Symmetric encryption (with session keys) is used for all actual data — it's much faster. |
| "The server's private key can decrypt past sessions" | Only if RSA key exchange was used (no forward secrecy). With Diffie-Hellman (TLS 1.3 default), session keys are ephemeral and can't be recovered. |
| "HTTPS makes websites slower" | Modern TLS (1.3) adds only **1 RTT** overhead. With session resumption, it can be **0 RTT**. The security benefit far outweighs this minimal latency. |

---

### Interview Takeaways — Module 7

1. **HTTPS = HTTP + TLS.** Provides confidentiality, integrity, and authentication.
2. **Symmetric encryption** = same key, fast (AES). **Asymmetric** = public/private key pair, slow (RSA, DH).
3. **Hybrid approach:** Asymmetric for key exchange → symmetric for data. Best of both.
4. **Certificate** binds domain + public key, signed by a CA. Browser verifies the chain of trust.
5. **CA verifies domain ownership** and signs certificates. Root CAs are pre-trusted by browsers/OS.
6. **TLS handshake:** Client Hello → Server Hello → Certificate → Key Exchange → Session Keys → Encrypted.
7. **Session keys** are per-connection symmetric keys derived from the Diffie-Hellman shared secret.
8. **TLS 1.3:** 1-RTT handshake, mandatory forward secrecy, 0-RTT resumption.
9. **Forward secrecy:** Ephemeral keys per session — compromising the server key doesn't expose past sessions.
10. **MITM is prevented** by certificate verification — attacker can't get a valid cert for your domain.
11. **Connection flow:** DNS → TCP handshake → TLS handshake → Encrypted HTTP.
12. **HTTPS does NOT encrypt:** domain name (SNI in TLS hello), DNS queries (separate protocol).
