# Module 6.6 — Email & File Protocols

> **GOOD TO KNOW — Minor module. Brief coverage for occasional interview questions.**

---

## 1. SMTP (Simple Mail Transfer Protocol) `GOOD TO KNOW`

**What:** The protocol for **sending** email.

**How it works:**
- Your email client (or server) connects to an SMTP server on **port 25** (or 587 for submission with auth).
- Client sends commands: `HELO`, `MAIL FROM`, `RCPT TO`, `DATA`, then the message body.
- The SMTP server relays the email to the recipient's mail server (via DNS MX record lookup).

**Key points:**
- SMTP is **push-only** — it sends/relays mail. It does NOT retrieve mail.
- SMTP is **text-based** (like HTTP/1.1).
- Modern SMTP uses **STARTTLS** for encryption.
- Email delivery chain: Sender → Sender's SMTP server → Recipient's SMTP server → Recipient's mailbox.

---

## 2. POP3 vs IMAP `GOOD TO KNOW`

These protocols **retrieve** email from the mail server.

| Feature | POP3 | IMAP |
|---------|------|------|
| **Full Name** | Post Office Protocol v3 | Internet Message Access Protocol |
| **Port** | 110 (995 with TLS) | 143 (993 with TLS) |
| **Model** | Download and delete | Sync and keep on server |
| **Server storage** | Emails removed after download | Emails stay on server |
| **Multi-device** | ❌ Poor (email on one device only) | ✅ Great (synced across all devices) |
| **Offline** | ✅ Works offline after download | Partially (can cache) |
| **Folders/Organization** | Local only | Server-side folders synced everywhere |
| **Modern usage** | Rare | Standard (Gmail, Outlook use IMAP) |

**Interview answer:** "POP3 downloads emails and removes them from the server — fine for a single device but doesn't sync. IMAP keeps emails on the server and syncs across multiple devices — it's the standard today. SMTP handles sending; POP3/IMAP handle receiving."

---

## 3. FTP (File Transfer Protocol) `GOOD TO KNOW`

**What:** A protocol for **transferring files** between client and server.

**Key facts:**
- Uses **two connections:** Control channel (port 21) for commands, Data channel (port 20) for file transfer.
- Supports upload, download, directory listing, rename, delete.
- **Not encrypted** by default — credentials sent in plaintext. SFTP (SSH + FTP) or FTPS (FTP + TLS) provide encryption.

### FTP vs HTTP for File Transfer

| Feature | FTP | HTTP |
|---------|-----|------|
| Purpose | Dedicated file transfer | General web communication |
| Connections | 2 (control + data) | 1 |
| Authentication | Built-in (username/password) | Via headers (cookies, tokens) |
| Resume downloads | Built-in | Range headers (partial support) |
| Modern usage | Legacy, declining | Standard (S3, CDN, REST APIs) |
| Firewall-friendly | ❌ (two connections, dynamic ports) | ✅ (single connection, port 80/443) |

**Interview point:** "FTP is rarely used in modern systems. File transfer is typically handled via HTTP (REST APIs, S3 presigned URLs, CDNs). FTP's two-connection model makes it firewall-unfriendly, and it lacks encryption by default."

---

## Interview Questions + Answers

---

**Q1: What is SMTP and what is it used for?**

**Ideal Answer:**
"SMTP is the Simple Mail Transfer Protocol — used for sending and relaying email. It's a push protocol on port 25 (or 587). When you send an email, your client sends it to your SMTP server, which looks up the recipient's mail server via DNS MX records and delivers it. SMTP only sends — POP3 or IMAP are used to retrieve email."

---

**Q2: What is the difference between POP3 and IMAP?**

**Ideal Answer:**
"POP3 downloads emails from the server and typically deletes them — the email exists on one device. IMAP syncs emails and keeps them on the server — accessible from multiple devices with folders synced across all of them. IMAP is the modern standard; POP3 is mostly legacy."

---

**Q3: What are the three email protocols and what does each do?**

**Ideal Answer:**
"SMTP sends and relays email (push). POP3 downloads email from the server (pull, download-and-delete). IMAP syncs email with the server (pull, keeps on server). A complete email flow: sender uses SMTP to send → recipient uses IMAP to read."

---

**Q4: Why is FTP rarely used today?**

**Ideal Answer:**
"FTP uses two connections (control on port 21, data on port 20), which makes it firewall-unfriendly. It sends credentials in plaintext by default. Modern file transfer uses HTTP (REST APIs, S3, CDNs), which is single-connection, encrypted with TLS, and works through any firewall."

---

### Common Mistakes

| Mistake | Correction |
|---------|------------|
| "SMTP retrieves email" | SMTP **sends** email. POP3/IMAP **retrieve** it. |
| "POP3 and IMAP are the same" | POP3 downloads and deletes. IMAP syncs and keeps on server. Very different for multi-device use. |
| "FTP is secure" | FTP is **plaintext** by default. SFTP (over SSH) or FTPS (over TLS) add encryption. |

### Interview Takeaways — Module 6.6

1. **SMTP** = sending email (port 25/587). **POP3** = download-and-delete (port 110). **IMAP** = sync-and-keep (port 143).
2. **IMAP** is the modern standard — syncs across devices. **POP3** is legacy.
3. **FTP** uses 2 connections, is plaintext, and is largely replaced by HTTP-based file transfer.
4. Email flow: Sender → SMTP → Recipient's SMTP server → IMAP/POP3 → Recipient.
