# Module 9 — Real-World Networking

> **This module connects everything together.**

---

## 1. What Happens When You Type `https://google.com` Into Your Browser? `MUST KNOW`

> This is the single most important interview question in all of Computer Networks.

### The Complete Journey

---

### Step 1: URL Parsing `[Application Layer]`

The browser parses the URL:
- **Scheme:** `https` → use TLS, port 443
- **Domain:** `google.com`
- **Path:** `/` (default)
- **Port:** 443 (default for HTTPS)

The browser checks its **HSTS list** — if `google.com` is on it, it forces HTTPS even if you typed `http://`.

---

### Step 2: DNS Lookup `[Application Layer, UDP Port 53]`

The browser needs to resolve `google.com` → IP address.

1. **Browser DNS cache** → miss
2. **OS DNS cache** → miss
3. OS queries **recursive resolver** (e.g., `8.8.8.8`) over UDP port 53
4. Resolver cache → miss
5. Resolver queries **root DNS** → "Ask `.com` TLD at `a.gtld-servers.net`"
6. Resolver queries **.com TLD** → "Ask Google's authoritative server at `ns1.google.com`"
7. Resolver queries **authoritative server** → "`google.com` = `142.250.190.46`"
8. Result cached at every level (with TTL)

**Protocol:** DNS over UDP. **Layer:** Application.

---

### Step 3: Determine Local vs Remote `[Network Layer]`

The OS compares `142.250.190.46` with the host's subnet (e.g., `192.168.1.0/24`).

`142.250.190.46` is **NOT** on the local network → must route through the **default gateway** (router).

---

### Step 4: ARP for Default Gateway `[Data Link Layer]`

The OS needs the router's **MAC address** to create an Ethernet frame.

- Check **ARP cache** for the gateway IP (e.g., `192.168.1.1`).
- If not cached: send **ARP broadcast** ("Who has 192.168.1.1?").
- Router responds with its MAC address.
- Cache the result.

**Protocol:** ARP (broadcast). **Layer:** Data Link (L2).

---

### Step 5: TCP 3-Way Handshake `[Transport Layer]`

```
Client (192.168.1.10:52431)                    Server (142.250.190.46:443)
  |                                               |
  |── SYN (seq=x) ──────────────────────────────>|
  |                                               |
  |<───────────────────────── SYN-ACK (seq=y, ack=x+1) ──|
  |                                               |
  |── ACK (seq=x+1, ack=y+1) ──────────────────>|
  |                                               |
  |         TCP Connection Established            |
```

- Client picks ephemeral port (52431), destination port 443.
- Each TCP segment is wrapped in an IP packet (src: `192.168.1.10`, dst: `142.250.190.46`).
- Each IP packet is wrapped in an Ethernet frame (dst MAC: router's MAC).
- **At each router hop:** IP stays same, MAC gets rewritten to next hop.
- Routers use **routing tables + longest prefix match** to forward.

**Protocol:** TCP. **Layer:** Transport (L4). **Cost:** 1 RTT.

---

### Step 6: TLS Handshake `[Security Layer]`

```
Client                                          Server
  |── Client Hello (TLS version, ciphers, random_c) ──>|
  |                                                      |
  |<── Server Hello (chosen cipher, random_s) ──────────|
  |<── Certificate (server's public key + CA signature) ─|
  |<── Server Key Exchange (DH parameters) ─────────────|
  |                                                      |
  | [Verify certificate: CA signature ✓, domain ✓, expiry ✓]
  |                                                      |
  |── Client Key Exchange (DH parameters) ─────────────>|
  |                                                      |
  | [Both derive shared secret → symmetric session keys] |
  |                                                      |
  |── Finished (encrypted) ────────────────────────────>|
  |<── Finished (encrypted) ─────────────────────────────|
  |                                                      |
  |         Encrypted Channel Established                |
```

- Asymmetric crypto (Diffie-Hellman) for key exchange.
- Symmetric session keys derived for all subsequent data.
- TLS 1.3: 1 RTT. TLS 1.2: 2 RTTs.

**Protocol:** TLS. **Layer:** Between Transport and Application.

---

### Step 7: HTTP Request `[Application Layer]`

Browser sends (encrypted by TLS):

```
GET / HTTP/2
Host: google.com
User-Agent: Mozilla/5.0 ...
Accept: text/html
Accept-Encoding: gzip, br
Cookie: NID=...
```

This HTTP request is:
- Encrypted by **TLS** → ciphertext
- Segmented by **TCP** → segments with sequence numbers
- Packetized by **IP** → packets with src/dst IP
- Framed by **Ethernet** → frames with src/dst MAC (to router)
- Sent as **bits** on the wire/air

---

### Step 8: Server-Side Processing

The request may traverse:

1. **CDN edge server** — if the content is cached nearby, respond immediately.
2. **Load balancer** — distributes request to one of many backend servers (round-robin, least connections, etc.).
3. **Reverse proxy** (Nginx) — may serve cached content or forward to application server.
4. **Application server** — processes the request, queries databases, generates HTML.
5. **HTTP Response** generated:

```
HTTP/2 200 OK
Content-Type: text/html; charset=UTF-8
Content-Encoding: gzip
Set-Cookie: NID=...; path=/; domain=.google.com
Cache-Control: private, max-age=0
```

---

### Step 9: HTTP Response Transmission `[All Layers]`

- Response is **encrypted** by TLS.
- **Segmented** by TCP (potentially many segments for a full web page).
- TCP ensures **reliable delivery** — ACKs, retransmission if needed.
- Packets traverse routers back to the client.
- **Flow control** prevents server from overwhelming client.
- **Congestion control** adapts to network conditions.

---

### Step 10: Browser Receives and Renders

1. TCP reassembles segments in order.
2. TLS decrypts the data.
3. Browser parses the HTTP response.
4. Parses HTML → discovers additional resources (CSS, JS, images, fonts).
5. Makes **additional HTTP requests** for each resource (reusing the TCP connection via **keep-alive** / HTTP/2 multiplexing).
6. Builds DOM, applies CSS, executes JavaScript.
7. **Renders the page.**

---

### Summary Diagram

```
[Browser]
    ↓ URL parsing
[DNS Lookup] ──── UDP:53 ──── Resolver → Root → TLD → Auth
    ↓ IP address obtained
[ARP] ──── Broadcast ──── Get router's MAC
    ↓ MAC address obtained
[TCP Handshake] ──── SYN/SYN-ACK/ACK ──── 1 RTT
    ↓ TCP connection
[TLS Handshake] ──── Hello/Cert/KeyExchange ──── 1-2 RTT
    ↓ Encrypted channel
[HTTP Request] ──── GET / ──── Encrypted
    ↓
[Server: CDN/LB/App] ──── Process request
    ↓
[HTTP Response] ──── 200 OK + HTML ──── Encrypted
    ↓
[Browser Renders Page]
```

**Total minimum latency before first byte:**
- DNS: ~0ms (cached) to ~100ms (full resolution)
- TCP: 1 RTT
- TLS 1.3: 1 RTT
- HTTP request + server processing: 1+ RTT
- **Minimum: ~2-3 RTTs** before the browser starts receiving content.

---

## 2. Real-World Infrastructure Concepts `GOOD TO KNOW`

### Proxy (Forward Proxy)
- Sits between **client** and the internet.
- Client sends requests to the proxy, proxy forwards to the destination.
- **Use cases:** Content filtering (corporate networks), caching, anonymity, bypassing geo-restrictions.
- The destination server sees the **proxy's IP**, not the client's.

### Reverse Proxy
- Sits in front of **servers**, invisible to the client.
- Client thinks it's talking directly to the server.
- **Use cases:**
  - **Load balancing** — distribute requests across backend servers.
  - **SSL termination** — handle TLS at the proxy, backend communicates in plaintext internally.
  - **Caching** — serve cached responses without hitting the backend.
  - **Security** — hide backend server details, act as a WAF.
- Examples: **Nginx, HAProxy, Cloudflare.**

### Proxy vs Reverse Proxy

| | Forward Proxy | Reverse Proxy |
|-|--------------|---------------|
| Position | Client side | Server side |
| Protects | Client identity | Server identity |
| Client aware? | Yes (client configured to use it) | No (client doesn't know about it) |
| Use case | Filtering, anonymity | Load balancing, caching, SSL |

### Load Balancer `MUST KNOW`
- Distributes incoming requests across multiple backend servers.
- **Why:** High availability (if one server dies, others handle traffic) + horizontal scaling.
- **Algorithms:**
  - **Round-robin:** Requests go to servers in rotation.
  - **Least connections:** Send to the server with fewest active connections.
  - **IP hash:** Same client IP always goes to the same server (session affinity).
- **L4 (TCP) load balancer:** Routes based on IP/port. Faster but can't inspect HTTP.
- **L7 (HTTP) load balancer:** Routes based on HTTP content (URL, headers, cookies). More flexible.

### CDN (Content Delivery Network) `MUST KNOW`
- Network of **geographically distributed servers** that cache content close to users.
- **Why:** Reduces latency by serving content from a nearby edge server instead of the origin.
- Serves **static content** (images, CSS, JS, videos) and sometimes dynamic content.
- Examples: **Cloudflare, AWS CloudFront, Akamai, Fastly.**
- How it works: First request → CDN fetches from origin, caches it. Subsequent requests → served from CDN edge.

### Firewall `GOOD TO KNOW`
- Monitors and filters network traffic based on rules.
- Can block by **IP address, port, protocol, or content**.
- **Stateful firewall:** Tracks connection state — only allows responses to established connections.
- **WAF (Web Application Firewall):** L7 firewall that inspects HTTP traffic — blocks SQL injection, XSS, etc.

### Caching at Various Levels

```
Browser Cache → CDN Cache → Reverse Proxy Cache → Application Cache → Database Cache
```

Each level reduces latency and load on the next level. Cache misses fall through to the next level.

### Connection Reuse
- **HTTP keep-alive:** Reuse TCP connection for multiple HTTP requests.
- **HTTP/2 multiplexing:** Multiple concurrent requests over one connection.
- **Connection pooling:** Backend servers maintain pools of database connections to avoid repeated connection setup.
- **Why it matters:** TCP handshake + TLS handshake = 2-3 RTTs overhead per new connection.

---

## Interview Questions + Answers

---

**Q1: What happens when you type https://google.com in your browser?**

**Ideal Answer:**
"The browser parses the URL and needs to resolve `google.com` to an IP address. It checks browser and OS DNS caches, then queries a recursive DNS resolver, which queries root → .com TLD → Google's authoritative DNS server, returning the IP. The OS checks if the destination is local or remote — it's remote, so it determines the default gateway. If the router's MAC isn't cached, ARP resolves it. Then a TCP 3-way handshake establishes a connection (1 RTT). A TLS handshake follows — exchanging cipher suites, verifying the server's certificate, and establishing symmetric session keys (1 more RTT with TLS 1.3). The browser sends an encrypted HTTP GET request. The request may hit a CDN, load balancer, or reverse proxy before reaching the application server. The server processes the request and sends back an encrypted HTTP response. TCP ensures reliable delivery. The browser decrypts the response, parses HTML, requests additional resources (CSS, JS, images) reusing the connection, and renders the page."

---

**Q2: What role does DNS play? What if the DNS server is down?**

**Ideal Answer:**
"DNS translates `google.com` to an IP address. If the DNS server is down, domain names can't be resolved and websites won't load — even though the servers are up. The OS typically has backup DNS servers configured. If the record is already cached (and TTL hasn't expired), it still works. You could also access the site directly by IP address, bypassing DNS."

---

**Q3: Why is ARP needed in this flow?**

**Ideal Answer:**
"The host needs to send the packet to the default gateway (router), but Ethernet frames require MAC addresses. ARP resolves the router's IP address to its MAC address. Without ARP, the host can't create the Ethernet frame to send to the router."

---

**Q4: What happens at the router level?**

**Ideal Answer:**
"The router receives the Ethernet frame, strips the L2 header, reads the destination IP from the IP header, consults its routing table using longest prefix match to find the next hop, decrements TTL, creates a new Ethernet frame with the next hop's MAC address, and sends it out. This repeats at each router until the packet reaches Google's network."

---

**Q5: What is a load balancer and where does it sit?**

**Ideal Answer:**
"A load balancer sits in front of backend servers and distributes incoming requests across them. It can operate at L4 (TCP — routes by IP/port) or L7 (HTTP — routes by URL, headers, cookies). It provides high availability — if one server fails, traffic goes to others — and enables horizontal scaling. Common algorithms include round-robin, least connections, and IP hash."

---

**Q6: What is a CDN and why is it useful?**

**Ideal Answer:**
"A CDN is a network of geographically distributed servers that cache content close to users. Instead of every request going to the origin server (which might be across the world), the CDN serves content from a nearby edge server, reducing latency significantly. It's especially useful for static assets like images, CSS, and JavaScript."

---

**Q7: What is the difference between a proxy and a reverse proxy?**

**Ideal Answer:**
"A forward proxy sits on the client side — clients send requests through it to access the internet. It hides the client's identity and is used for filtering, caching, and anonymity. A reverse proxy sits on the server side — clients connect to it thinking it's the actual server. It hides backend servers and is used for load balancing, SSL termination, caching, and security. The client configures a forward proxy explicitly; a reverse proxy is transparent to the client."

---

**Q8: What is the difference between L4 and L7 load balancing?**

**Ideal Answer:**
"L4 load balancing operates at the transport layer — it routes based on IP addresses and port numbers without inspecting the actual HTTP content. It's fast but less flexible. L7 load balancing operates at the application layer — it can route based on URL paths, HTTP headers, cookies, and request content. It's more flexible (e.g., route /api to one server group and /static to another) but slightly slower due to content inspection."

---

**Q9: Where does caching happen in this flow?**

**Ideal Answer:**
"Caching happens at multiple levels: the browser caches DNS results and HTTP responses, the OS caches DNS, the DNS resolver caches query results, CDN edge servers cache static content, reverse proxies cache HTTP responses, and application servers may cache database queries. Each level reduces latency and load on deeper layers."

---

**Q10: What happens if a packet is lost during this flow?**

**Ideal Answer:**
"TCP handles it. The receiver detects the missing segment (gap in sequence numbers) and sends duplicate ACKs. After 3 duplicate ACKs, the sender performs fast retransmit. If no ACKs come at all, the sender's retransmission timer expires and it retransmits, also reducing its congestion window. The application is unaware — TCP handles reliability transparently."

---

**Q11: Why does HTTPS need both TCP and TLS handshakes?**

**Ideal Answer:**
"They solve different problems. TCP handshake establishes a reliable transport connection — ensures both sides can communicate and synchronizes sequence numbers. TLS handshake establishes security — authenticates the server, negotiates encryption, and derives session keys. You can't do TLS without a transport connection first, and TCP alone doesn't provide any security."

---

**Q12: How does the browser know to reuse the TCP connection?**

**Ideal Answer:**
"HTTP/1.1 uses persistent connections (keep-alive) by default — the TCP connection stays open after the first request/response. HTTP/2 goes further with multiplexing — multiple requests and responses flow concurrently over a single TCP connection. The browser reuses the connection for subsequent requests to the same server, avoiding the overhead of repeated TCP + TLS handshakes."

---

### Common Mistakes

| Mistake | Correction |
|---------|------------|
| "The browser connects directly to Google's server at Layer 2" | The Ethernet frame goes to the **default gateway** (router). The frame's destination MAC is the router's, not Google's. |
| "DNS is part of the TCP connection" | DNS is a **separate** query (typically UDP) that happens **before** the TCP handshake. |
| "TLS replaces TCP" | TLS runs **on top of** TCP. You need TCP first, then TLS, then HTTP. |
| "The entire URL is encrypted by HTTPS" | The **domain name** is visible in DNS queries and in TLS SNI (Server Name Indication). The path, headers, and body are encrypted. |
| "A CDN replaces the origin server" | A CDN **caches** content and serves it when possible. Cache misses still go to the origin server. |
| "Load balancers are only for large-scale systems" | Even small deployments benefit from load balancers for **high availability** — if one server dies, traffic fails over. |

---

### Interview Takeaways — Module 9

1. **The URL flow:** DNS → ARP (if needed) → TCP handshake → TLS handshake → HTTP request → Server processing → HTTP response → Render.
2. **DNS is separate** from the main connection — happens first over UDP port 53.
3. **ARP resolves the gateway's MAC** — not the destination server's MAC.
4. **TCP + TLS combined:** 2-3 RTTs before any HTTP data flows.
5. **Server-side path:** CDN → Load Balancer → Reverse Proxy → Application Server.
6. **CDN** caches content at edge locations close to users → reduces latency.
7. **Load balancer** distributes requests for high availability and scaling. L4 (IP/port) vs L7 (HTTP content).
8. **Forward proxy** hides clients. **Reverse proxy** hides servers.
9. **Caching everywhere:** Browser, OS, DNS resolver, CDN, reverse proxy, application, database.
10. **Connection reuse** (keep-alive, HTTP/2 multiplexing) avoids repeated handshake overhead.
11. **HTTPS doesn't encrypt:** domain name (DNS + SNI), IP addresses. It encrypts: path, headers, body.
