# Module 5 — DNS

---

## 1. What DNS Is `MUST KNOW`

**DNS = Domain Name System** — translates human-readable domain names (like `google.com`) into IP addresses (like `142.250.190.46`).

**Why it exists:** Humans remember names, computers use IP addresses. Without DNS, you'd need to memorize IP addresses for every website.

**Key facts:**
- DNS is an **application-layer** protocol.
- Uses **UDP port 53** (for queries) and **TCP port 53** (for zone transfers and large responses > 512 bytes).
- DNS is a **distributed, hierarchical** database — no single server has all the answers.

---

## 2. DNS Hierarchy `MUST KNOW`

```
                        . (Root)
                       / | \
                     /   |   \
                  .com  .org  .net  .io  ...    ← TLD Servers
                 / | \
               /   |   \
          google  amazon  github    ...         ← Authoritative Servers
           |
      www  mail  maps  ...                      ← Individual Records
```

### Root DNS Servers
- 13 logical root server groups (a.root-servers.net through m.root-servers.net).
- They don't know the final IP — they know **where TLD servers are**.
- Anycast — each "root server" is actually hundreds of physical servers worldwide.

### TLD (Top-Level Domain) Servers
- Handle `.com`, `.org`, `.net`, `.io`, `.edu`, etc.
- They don't know the final IP — they know **where authoritative servers are** for each domain under them.

### Authoritative DNS Servers
- Hold the **actual DNS records** for a domain.
- Example: Google's authoritative server knows that `google.com → 142.250.190.46`.
- This is the final stop in the DNS hierarchy.

### Recursive DNS Resolver
- The resolver your computer contacts first (ISP's resolver, or public ones like `8.8.8.8` / `1.1.1.1`).
- **Does all the work** — queries root → TLD → authoritative on your behalf.
- Caches results to speed up future queries.

---

## 3. DNS Resolution — How `google.com` Becomes an IP Address `MUST KNOW`

**Step-by-step flow:**

```
Browser                    OS                Recursive Resolver      Root      .com TLD    google.com Auth
  |                        |                       |                  |           |              |
  |-- Check browser cache  |                       |                  |           |              |
  |   (miss)               |                       |                  |           |              |
  |-- Query OS ----------->|                       |                  |           |              |
  |                        |-- Check OS cache      |                  |           |              |
  |                        |   (miss)              |                  |           |              |
  |                        |-- Query resolver ---->|                  |           |              |
  |                        |                       |-- Check cache    |           |              |
  |                        |                       |   (miss)         |           |              |
  |                        |                       |-- Query root --->|           |              |
  |                        |                       |<- "Ask .com TLD" |           |              |
  |                        |                       |                  |           |              |
  |                        |                       |-- Query .com TLD ---------->|              |
  |                        |                       |<- "Ask google.com auth" ----|              |
  |                        |                       |                  |           |              |
  |                        |                       |-- Query auth ------------------------------>|
  |                        |                       |<- "142.250.190.46" -------------------------|
  |                        |                       |                  |           |              |
  |                        |<-- 142.250.190.46 ----|                  |           |              |
  |<-- 142.250.190.46 -----|                       |                  |           |              |
  |                        |                       |                  |           |              |
  [Cache it, connect]      [Cache it]              [Cache it]
```

**Detailed steps:**

1. **Browser checks its own DNS cache.** If found → use it.
2. **OS checks its DNS cache.** If found → use it.
3. OS sends query to the **configured recursive resolver** (e.g., `8.8.8.8`).
4. **Resolver checks its cache.** If found and TTL hasn't expired → return immediately.
5. Resolver queries a **root server** → root says "I don't know `google.com`, but `.com` TLD is at `a.gtld-servers.net`."
6. Resolver queries the **.com TLD server** → TLD says "I don't know `google.com`'s IP, but its authoritative server is `ns1.google.com`."
7. Resolver queries **Google's authoritative server** → gets the answer: `142.250.190.46`.
8. Resolver **caches** the result (respecting TTL) and returns it to the OS.
9. OS caches it, returns to browser.
10. Browser caches it, connects to the IP.

---

## 4. Recursive vs Iterative Queries `MUST KNOW`

| Type | How It Works | Who Does the Work | Used Between |
|------|-------------|-------------------|-------------|
| **Recursive** | "Give me the full answer" — resolver does all the work | Resolver | Client → Resolver |
| **Iterative** | "I don't know, try asking X" — referrals | Each server only answers what it knows | Resolver → Root → TLD → Auth |

**Interview answer:** "A client sends a recursive query to its resolver — the resolver is expected to return the final answer. The resolver then sends iterative queries to root, TLD, and authoritative servers — each one responds with a referral to the next server. So the client makes one recursive call; the resolver makes multiple iterative calls."

---

## 5. DNS Caching `MUST KNOW`

Caching happens at **every level:**
- Browser DNS cache
- OS DNS cache
- Recursive resolver cache
- Even intermediate servers can cache

### DNS TTL (Time to Live)
- Every DNS record has a TTL (in seconds) — how long it can be cached before it must be refreshed.
- **Low TTL** (e.g., 60s): Frequent lookups, but changes propagate fast. Good during migrations.
- **High TTL** (e.g., 86400s = 24h): Fewer lookups, but changes are slow to propagate.

**Why caching matters:**
- Reduces latency (no need to traverse the entire hierarchy every time).
- Reduces load on root/TLD/authoritative servers.
- Most DNS lookups are served from cache.

---

## 6. DNS Records `MUST KNOW`

| Record | Purpose | Example |
|--------|---------|---------|
| **A** | Maps domain → **IPv4** address | `google.com → 142.250.190.46` |
| **AAAA** | Maps domain → **IPv6** address | `google.com → 2607:f8b0:4004:800::200e` |
| **CNAME** | **Alias** — maps domain → another domain | `www.example.com → example.com` |
| **MX** | **Mail exchange** — specifies mail server | `example.com → mail.example.com (priority 10)` |
| **NS** | **Name server** — authoritative DNS servers for domain | `google.com → ns1.google.com` |
| **TXT** | Arbitrary text — used for SPF, DKIM, domain verification | `v=spf1 include:_spf.google.com ~all` |

**Interview points:**
- **CNAME** cannot coexist with other records for the same name. `www.example.com CNAME example.com` means any query for `www.example.com` first resolves `example.com`.
- **MX** records have priority values — lower number = higher priority.
- **TXT** records are commonly used for email authentication (SPF/DKIM) and domain ownership verification (Google, AWS).

---

## 7. DNS over HTTPS (DoH) and DNS over TLS (DoT) `GOOD TO KNOW`

**Problem:** Traditional DNS is **unencrypted** (plaintext over UDP port 53). Anyone on the network (ISP, attacker) can see your DNS queries — they know what websites you're visiting.

| Method | How | Port | Benefit |
|--------|-----|------|---------|
| **DoH** (DNS over HTTPS) | DNS queries sent over HTTPS | 443 | Encrypted + looks like regular web traffic (hard to block) |
| **DoT** (DNS over TLS) | DNS queries sent over TLS | 853 | Encrypted (but uses dedicated port, easier to identify/block) |

**Why they exist:** Privacy — prevent ISPs or attackers from snooping on DNS queries.

---

## Interview Questions + Answers

---

**Q1: What is DNS and why does it exist?**

**Ideal Answer:**
"DNS is the Domain Name System — it translates human-readable domain names like `google.com` into IP addresses that computers use for routing. It exists because humans remember names, not IP addresses. DNS is a distributed, hierarchical system — no single server has all the mappings."

---

**Q2: Explain how DNS resolution works.**

**Ideal Answer:**
"When you type `google.com`, the browser checks its DNS cache, then the OS cache. If not found, the OS queries a recursive resolver (like 8.8.8.8). The resolver checks its cache, and if needed, sends iterative queries: first to a root server (which says 'ask the .com TLD'), then to the .com TLD (which says 'ask Google's authoritative server'), then to the authoritative server, which returns the actual IP. The resolver caches the result and returns it to the client."

---

**Q3: What is the difference between recursive and iterative DNS queries?**

**Ideal Answer:**
"In a recursive query, the client expects the resolver to return the final answer — the resolver does all the legwork. In an iterative query, the server responds with a referral ('I don't know, try asking X'). Typically, your computer sends a recursive query to the resolver, and the resolver sends iterative queries to root, TLD, and authoritative servers."

---

**Q4: What are the main DNS record types?**

**Ideal Answer:**
"A records map a domain to an IPv4 address. AAAA maps to IPv6. CNAME creates an alias — maps one domain to another. MX specifies the mail server for a domain. NS specifies the authoritative name servers. TXT holds arbitrary text, commonly used for email authentication (SPF/DKIM) and domain verification."

---

**Q5: What is DNS caching and why is TTL important?**

**Ideal Answer:**
"DNS results are cached at every level — browser, OS, and resolver — to avoid repeating the full resolution for every query. TTL specifies how long a record can be cached. Low TTL means changes propagate quickly but causes more DNS queries. High TTL reduces queries but means changes take longer to propagate. During a migration, you'd lower the TTL first so the switch happens faster."

---

**Q6: Why does DNS primarily use UDP?**

**Ideal Answer:**
"DNS queries and responses are typically small (fit in a single UDP datagram under 512 bytes), so the overhead of establishing a TCP connection for each query would be wasteful. UDP is faster — no handshake needed. DNS falls back to TCP for responses larger than 512 bytes (with EDNS, up to ~4096) and for zone transfers between DNS servers."

> **Follow-up Q: When does DNS use TCP?**
>
> **Ideal Answer:** "DNS uses TCP for zone transfers (replicating records between servers) and when a response is too large for UDP. Also, DNS over TLS (DoT) uses TCP as its transport."

---

**Q7: What happens if a DNS server is unavailable?**

**Ideal Answer:**
"If the recursive resolver is down, DNS queries fail and domain names can't be resolved — websites won't load even though the servers are up. The OS typically has multiple DNS servers configured and will try the next one. If the authoritative server is down, cached records still work until their TTL expires. Most domains have multiple authoritative servers (NS records) for redundancy."

---

**Q8: What is a CNAME record? When would you use it?**

**Ideal Answer:**
"A CNAME is an alias — it maps one domain name to another. For example, `www.example.com CNAME example.com` makes `www.example.com` resolve to whatever `example.com` resolves to. It's useful when you want multiple names to point to the same destination, so you only need to update one A record. A CNAME cannot coexist with other record types for the same name."

---

**Q9: Can DNS be a security vulnerability?**

**Ideal Answer:**
"Yes. DNS spoofing or cache poisoning — an attacker injects false DNS records into a resolver's cache, redirecting users to malicious sites. Also, traditional DNS is unencrypted, so attackers can see what domains you're querying. DNS over HTTPS/TLS addresses the privacy issue. DNSSEC (DNS Security Extensions) addresses spoofing by adding cryptographic signatures to DNS records."

---

**Q10: You changed a DNS record but users still see the old IP. Why?**

**Ideal Answer:**
"DNS caching. The old record is still cached at various levels — browser, OS, recursive resolver — and the TTL hasn't expired yet. Users will see the old IP until the cached entries expire. This is why it's best practice to lower the TTL well in advance of a planned DNS change, so the transition happens faster."

---

### Common Mistakes

| Mistake | Correction |
|---------|------------|
| "DNS resolves domain names to MAC addresses" | DNS resolves domain names to **IP addresses**. ARP resolves IP to MAC. |
| "There is one root DNS server" | There are **13 logical root server groups**, each with many physical servers worldwide (anycast). |
| "DNS always uses UDP" | DNS primarily uses UDP, but falls back to **TCP** for large responses and zone transfers. |
| "DNS over HTTPS encrypts the website content" | DoH encrypts the **DNS query** only. The website content is encrypted by TLS/HTTPS separately. |
| "Changing a DNS record takes effect immediately" | It takes effect after **cached entries expire** (based on TTL). Propagation can take minutes to hours. |

---

### Interview Takeaways — Module 5

1. **DNS = domain name → IP address.** Distributed hierarchical database.
2. **Hierarchy: Root → TLD → Authoritative.** Recursive resolver does all the work for the client.
3. **Recursive query** = client asks resolver for final answer. **Iterative query** = server gives referrals.
4. **Caching at every level** (browser, OS, resolver). TTL controls how long records are cached.
5. **Key records:** A (IPv4), AAAA (IPv6), CNAME (alias), MX (mail), NS (nameserver), TXT (text/verification).
6. **DNS uses UDP port 53** for queries (fast, small). TCP for zone transfers and large responses.
7. **DNS change propagation** depends on TTL — lower TTL before migrations.
8. **DoH/DoT** encrypt DNS queries for privacy. Traditional DNS is plaintext.
9. **CNAME** is an alias to another domain. Cannot coexist with other records for the same name.
10. **DNS failure** = can't resolve names = websites don't load (even if servers are up).
