# Final CN Revision Sheet

> **Your go-to document for last-minute interview revision.**

---

## 1. Must-Remember Concepts

1. **TCP is connection-oriented and reliable; UDP is connectionless and unreliable.**
2. **MAC addresses change at every hop; IP addresses stay the same end-to-end (ignoring NAT).**
3. **The TLS handshake uses asymmetric encryption to exchange a symmetric session key.**
4. **The 3-way handshake (SYN → SYN-ACK → ACK) synchronizes sequence numbers between both sides.**
5. **Flow control prevents overwhelming the receiver (rwnd). Congestion control prevents overwhelming the network (cwnd). They are different.**
6. **Actual sending window = min(rwnd, cwnd).**
7. **A 5-tuple (src IP, src port, dst IP, dst port, protocol) uniquely identifies a connection.**
8. **DNS translates domain names to IP addresses. It's a separate UDP query before any TCP/TLS/HTTP happens.**
9. **HTTP is stateless. Cookies and sessions maintain state externally.**
10. **HTTPS = HTTP + TLS. Provides confidentiality, integrity, and authentication.**
11. **Routers operate at L3 (IP), switches at L2 (MAC), hubs at L1 (bits).**
12. **Longest prefix match: the most specific route wins when multiple routes match.**
13. **ARP resolves IP → MAC on the local network. It's broadcast-based and local only.**
14. **When sending to a remote host, ARP resolves the default gateway's MAC — NOT the destination's MAC.**
15. **TCP detects loss via timeout (severe → cwnd=1) or 3 dup ACKs (fast retransmit → cwnd halved).**
16. **TCP termination takes 4 steps because TCP is full-duplex — each direction closes independently.**
17. **TIME_WAIT (2×MSL) ensures the final ACK arrives and old packets expire.**
18. **HTTP/2 multiplexes streams over one TCP connection. HTTP/3 uses QUIC/UDP for true per-stream independence.**
19. **A certificate binds a domain to a public key, signed by a CA. The browser verifies the chain of trust.**
20. **NAT/PAT lets multiple private IPs share one public IP using port numbers.**
21. **CDN caches content at edge locations close to users to reduce latency.**
22. **Encapsulation: each layer wraps the above layer's output as its payload.**
23. **Ethernet MTU = 1500 bytes. Packets larger than this must be fragmented.**
24. **TTL prevents infinite routing loops — decremented at each hop, packet dropped at 0.**
25. **A server handles many clients on one port because each connection has a unique 5-tuple.**

---

## 2. Important Comparisons

### TCP vs UDP

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | Reliable (ACK, retransmission) | Unreliable (best-effort) |
| Ordering | Ordered (sequence numbers) | No ordering |
| Flow Control | Yes (rwnd) | No |
| Congestion Control | Yes (cwnd, slow start) | No |
| Speed | Slower (overhead) | Faster |
| Header Size | 20–60 bytes | 8 bytes |
| Data Model | Byte stream | Datagram/message |
| Use Cases | HTTP, SSH, email, file transfer | DNS, streaming, gaming, VoIP, QUIC |

### HTTP vs HTTPS

| Feature | HTTP | HTTPS |
|---------|------|-------|
| Port | 80 | 443 |
| Encryption | None (plaintext) | TLS (encrypted) |
| Confidentiality | ❌ | ✅ |
| Integrity | ❌ | ✅ |
| Authentication | ❌ | ✅ (certificates) |
| Performance | Slightly faster (no TLS) | ~1 RTT extra (TLS 1.3) |

### HTTP/1.1 vs HTTP/2 vs HTTP/3

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------|----------|--------|--------|
| Transport | TCP | TCP | QUIC (UDP) |
| Format | Text | Binary | Binary |
| Multiplexing | No | Yes (streams over 1 TCP) | Yes (independent streams) |
| HOL Blocking | HTTP-level | TCP-level remains | None |
| Header Compression | No | HPACK | QPACK |
| Connection Setup | 2-3 RTT | 2-3 RTT | 1 RTT (0-RTT resumption) |

### Switch vs Router

| Feature | Switch | Router |
|---------|--------|--------|
| Layer | L2 (Data Link) | L3 (Network) |
| Address Used | MAC address | IP address |
| Function | Connects devices within LAN | Connects different networks |
| Broadcast | Forwards broadcasts | Blocks broadcasts (boundary) |
| Table | MAC address table | Routing table |

### MAC vs IP vs Port

| | MAC | IP | Port |
|-|-----|-----|------|
| Layer | L2 | L3 | L4 |
| Size | 48 bits | 32 bits (IPv4) | 16 bits |
| Scope | Local (hop-by-hop) | End-to-end | Process on a host |
| Changes per hop? | Yes | No (except NAT) | No |
| Identifies | Device on local link | Device across internet | Process/service |

### Flow Control vs Congestion Control

| | Flow Control | Congestion Control |
|-|---|---|
| Prevents overwhelming | **Receiver** | **Network** |
| Controlled by | Receiver (advertises rwnd) | Sender (adjusts cwnd) |
| Variable | rwnd (receiver window) | cwnd (congestion window) |
| Mechanism | Window size in TCP header | Slow start, AIMD, fast recovery |

### Symmetric vs Asymmetric Encryption

| | Symmetric | Asymmetric |
|-|-----------|------------|
| Keys | Same key for encrypt/decrypt | Public key + private key |
| Speed | Fast (AES, ChaCha20) | Slow (RSA, ECDHE) |
| Key Problem | How to share securely? | Solved (public key is public) |
| Used For | Bulk data encryption | Key exchange, digital signatures |
| In TLS | Session key encryption | Handshake key exchange |

### Recursive vs Iterative DNS

| | Recursive | Iterative |
|-|-----------|-----------|
| "Give me the answer" | Yes — resolver returns final result | No — server gives referral |
| Who does the work | Resolver | Client follows referrals |
| Used between | Client → Resolver | Resolver → Root/TLD/Auth |

### Hub vs Switch

| | Hub | Switch |
|-|-----|--------|
| Layer | L1 (Physical) | L2 (Data Link) |
| Intelligence | None (repeats signal) | Learns MACs, forwards selectively |
| Collision Domain | 1 shared | 1 per port |
| Modern use | Obsolete | Standard |

### GET vs POST

| | GET | POST |
|-|-----|------|
| Purpose | Retrieve data | Create/submit data |
| Body | No | Yes |
| Idempotent | Yes | No |
| Cacheable | Yes | No |
| Safe | Yes | No |

### Public IP vs Private IP

| | Public | Private |
|-|--------|---------|
| Routable | Internet-routable | NOT routable on internet |
| Uniqueness | Globally unique | Reused in different networks |
| Assignment | ISP/registry | Anyone (internal use) |
| Ranges | All except private | 10.x, 172.16-31.x, 192.168.x |
| NAT needed? | No | Yes (to reach internet) |

### LAN vs WAN

| | LAN | WAN |
|-|-----|-----|
| Scope | Small area (office, home) | Large area (cities, countries) |
| Speed | High (1 Gbps+) | Lower |
| Latency | Low | Higher |
| Devices | Switches | Routers |
| Example | Office network | The internet |

### Circuit Switching vs Packet Switching

| | Circuit Switching | Packet Switching |
|-|-------------------|------------------|
| Path | Dedicated, reserved | Shared, per-packet |
| Efficiency | Wastes bandwidth during idle | Efficient sharing |
| Delay | Constant | Variable |
| Example | Telephone (PSTN) | Internet |
| Reliability | Guaranteed path | Packets can be lost/reordered |
---

### Distance Vector vs Link State Routing

| | Distance Vector (RIP) | Link State (OSPF) |
|-|---|---|
| Knowledge | Only neighbors' tables | Complete network topology |
| Algorithm | Bellman-Ford | Dijkstra's |
| Convergence | Slow (count-to-infinity) | Fast |
| Updates | Entire table, periodic | Only changes, event-driven |
| Metric | Hop count (max 16) | Link cost (bandwidth) |

### Stop-and-Wait vs Go-Back-N vs Selective Repeat

| | Stop-and-Wait | Go-Back-N | Selective Repeat |
|-|---|---|---|
| Window | 1 | N | N |
| On loss | Retransmit that frame | Retransmit from lost onward | Retransmit only lost frame |
| Receiver | Simple | Simple (no buffer) | Buffers out-of-order |
| Efficiency | Very low | Medium | High (≈ TCP) |

### WebSocket vs SSE vs Long Polling

| | Long Polling | SSE | WebSocket |
|-|---|---|---|
| Direction | Server → Client (simulated) | Server → Client | Bidirectional |
| Protocol | HTTP | HTTP | WebSocket (starts HTTP) |
| Data format | Any | Text only | Text + Binary |
| Auto-reconnect | Manual | Built-in | Manual |
| Overhead | High | Low | Very low |
| Best for | Fallback | Server push (feeds) | Bidirectional (chat, gaming) |

### REST vs RPC vs gRPC

| | REST | gRPC |
|-|---|---|
| Style | Resource-oriented (nouns) | Action-oriented (verbs) |
| Format | JSON (text) | Protobuf (binary) |
| Transport | HTTP/1.1 or HTTP/2 | HTTP/2 |
| Streaming | Not native | Built-in |
| Browser | Native | Needs gRPC-Web proxy |
| Best for | Public APIs, web | Internal microservices |

### POP3 vs IMAP

| | POP3 | IMAP |
|-|---|---|
| Model | Download and delete | Sync and keep on server |
| Multi-device | ❌ Poor | ✅ Synced |
| Port | 110 (995 TLS) | 143 (993 TLS) |
| Modern usage | Legacy | Standard |

### Classful vs Classless (CIDR) Addressing

| | Classful | Classless (CIDR) |
|-|---|---|
| Subnet sizes | Fixed: /8, /16, /24 only | Any prefix: /25, /26, /27... |
| Flexibility | None | Full |
| Waste | Severe (300 hosts → /16 = 65K) | Minimal (300 hosts → /23 = 510) |
| Modern usage | Obsolete | Standard |

### Repeater vs Hub vs Bridge vs Switch vs Router vs Gateway

| Device | Layer | Addresses | Function |
|--------|-------|-----------|----------|
| Repeater | L1 | None | Amplifies signal |
| Hub | L1 | None | Multi-port repeater, floods all |
| Bridge | L2 | MAC | Connects 2 segments, filters by MAC |
| Switch | L2 | MAC | Multi-port bridge, learns + forwards |
| Router | L3 | IP | Forwards between networks, routing table |
| Gateway | L7 | All | Protocol translation |

---

## 3. Important Protocols Table

| Protocol | Purpose | Layer | Important Interview Point |
|----------|---------|-------|--------------------------|
| **TCP** | Reliable, ordered transport | L4 | 3-way handshake, flow/congestion control, retransmission |
| **UDP** | Fast, unreliable transport | L4 | No handshake, 8-byte header, used by DNS/streaming/QUIC |
| **IP** | Logical addressing, routing | L3 | 32-bit (v4) / 128-bit (v6), routing via longest prefix match |
| **ICMP** | Error reporting, diagnostics | L3 | Used by ping (Echo) and traceroute (Time Exceeded) |
| **ARP** | IP → MAC resolution | L2/L3 | Broadcast request, unicast reply, local network only |
| **DNS** | Domain → IP resolution | L7 | Hierarchical, UDP:53, caching with TTL |
| **HTTP** | Web communication | L7 | Stateless, methods (GET/POST/PUT/DELETE), status codes |
| **TLS** | Encryption, authentication | Between L4-L7 | Certificates, handshake, symmetric session keys |
| **QUIC** | Reliable transport over UDP | L4 | Used by HTTP/3, per-stream reliability, 0-RTT |
| **Ethernet** | LAN framing, local delivery | L2 | MAC addressing, MTU=1500, CRC error detection |
| **DHCP** | Automatic IP assignment | L7 | Assigns IP, subnet mask, gateway, DNS to hosts |
| **NAT/PAT** | Private → public IP translation | L3 | Enables multiple private IPs to share one public IP |
| **SMTP** | Sending/relaying email | L7 | Push only (port 25/587). POP3/IMAP retrieve email |
| **POP3** | Retrieving email (download & delete) | L7 | Port 110. Legacy — replaced by IMAP |
| **IMAP** | Retrieving email (sync & keep) | L7 | Port 143. Modern standard — multi-device sync |
| **FTP** | File transfer | L7 | 2 connections (control:21, data:20). Largely replaced by HTTP |
| **gRPC** | Remote procedure calls | L7 | Protobuf + HTTP/2. Binary, fast, streaming. Internal services |
| **WebSocket** | Full-duplex bidirectional communication | L7 | HTTP upgrade → persistent TCP. Chat, gaming, real-time |

---

## 4. Important Flows

### ARP Resolution
1. Host checks ARP cache for destination IP → miss.
2. Host sends ARP Request broadcast: "Who has IP X?"
3. Target responds with ARP Reply (unicast): "I'm X, my MAC is Y."
4. Host caches mapping and sends data frame.

### DNS Lookup (Full Resolution)
1. Browser cache → OS cache → miss.
2. OS queries recursive resolver (UDP:53).
3. Resolver cache → miss.
4. Resolver → Root server → referral to .com TLD.
5. Resolver → .com TLD → referral to authoritative server.
6. Resolver → Authoritative → returns IP address.
7. Cached at all levels with TTL.

### TCP 3-Way Handshake
1. **Client → Server:** SYN, seq=x
2. **Server → Client:** SYN-ACK, seq=y, ack=x+1
3. **Client → Server:** ACK, seq=x+1, ack=y+1
4. Connection ESTABLISHED. Cost: 1 RTT.

### TCP 4-Way Termination
1. **A → B:** FIN (A done sending)
2. **B → A:** ACK (acknowledges FIN)
3. **B → A:** FIN (B done sending)
4. **A → B:** ACK (acknowledges FIN)
5. A enters TIME_WAIT (2×MSL).

### TLS Handshake (Simplified)
1. **Client Hello:** TLS versions, cipher suites, random.
2. **Server Hello:** Chosen cipher, random, certificate.
3. **Certificate verification** by client (CA chain, domain, expiry).
4. **Key exchange** (Diffie-Hellman) → shared secret.
5. **Session keys** derived → both send encrypted Finished.
6. All subsequent data encrypted with symmetric session keys.

### HTTP Request-Response
1. Client sends: `METHOD /path HTTP/1.1` + headers + body.
2. Server processes request.
3. Server sends: `HTTP/1.1 STATUS_CODE Reason` + headers + body.

### Complete HTTPS Request
1. **DNS:** Resolve domain → IP (UDP:53).
2. **TCP:** 3-way handshake (1 RTT).
3. **TLS:** Handshake — certificate, key exchange, session keys (1-2 RTT).
4. **HTTP:** Encrypted request sent, server processes, encrypted response returned.
5. **Browser:** Decrypts, parses HTML, fetches resources, renders.

### WebSocket Handshake (HTTP Upgrade)
1. Client sends HTTP GET with `Upgrade: websocket`, `Connection: Upgrade` headers.
2. Server responds with **101 Switching Protocols**.
3. TCP connection upgrades from HTTP to WebSocket protocol.
4. Full-duplex communication — both sides send messages with minimal framing (2-14 byte headers).

---

## 5. Top 50 CN Interview Questions

### Networking Basics (1–8)

**1. What is the OSI model?**
A: 7-layer reference model — Physical, Data Link, Network, Transport, Session, Presentation, Application. TCP/IP (4 layers) is what the internet actually uses.

**2. What is encapsulation?**
A: Each layer adds its header to the data from the layer above. Segment (L4) → Packet (L3) → Frame (L2) → Bits (L1). Reverse on receiving side.

**3. Difference between MAC, IP, and port?**
A: MAC (48-bit, L2) = local delivery, changes per hop. IP (32-bit, L3) = end-to-end routing, stays same. Port (16-bit, L4) = identifies process.

**4. Hub vs switch vs router?**
A: Hub = L1, repeats to all ports. Switch = L2, forwards by MAC. Router = L3, forwards by IP, separates broadcast domains.

**5. What is the 5-tuple?**
A: (src IP, src port, dst IP, dst port, protocol). Uniquely identifies every network connection.

**6. Bandwidth vs throughput vs latency?**
A: Bandwidth = max capacity. Throughput = actual rate achieved ≤ bandwidth. Latency = time delay.

**7. Circuit switching vs packet switching?**
A: Circuit = dedicated path, wasteful. Packet = shared, efficient, used by the internet. Tradeoff: needs TCP for reliability.

**8. Collision domain vs broadcast domain?**
A: Collision = where collisions happen (switch gives 1 per port). Broadcast = where broadcasts reach (router separates them).

### TCP/UDP (9–24)

**9. What is TCP?**
A: Connection-oriented, reliable, ordered transport protocol with flow/congestion control. Uses 3-way handshake.

**10. What is UDP?**
A: Connectionless, unreliable, fast transport protocol. 8-byte header. Used for DNS, streaming, gaming, QUIC.

**11. TCP vs UDP?**
A: TCP = reliable/ordered/slow. UDP = unreliable/unordered/fast. TCP for HTTP. UDP for real-time apps.

**12. Explain the 3-way handshake.**
A: SYN(seq=x) → SYN-ACK(seq=y, ack=x+1) → ACK(ack=y+1). Synchronizes sequence numbers. 1 RTT.

**13. Why 3-way, not 2-way?**
A: Both sides must confirm each other's ISN. 2-way can't confirm the server's ISN was received by the client.

**14. What is flow control?**
A: Prevents sender from overwhelming receiver. Receiver advertises rwnd (buffer space). Sender limits to rwnd.

**15. What is congestion control?**
A: Prevents sender from overwhelming network. Sender maintains cwnd. Slow start (exponential) → congestion avoidance (linear). 3 dup ACKs → halve cwnd. Timeout → cwnd=1.

**16. Flow control vs congestion control?**
A: Flow = receiver limit (rwnd). Congestion = network limit (cwnd). Different problems, different mechanisms.

**17. How does TCP detect packet loss?**
A: Timeout (severe) or 3 duplicate ACKs (fast retransmit). Timeout → cwnd=1, slow start. 3 dup ACKs → cwnd halved, fast recovery.

**18. What is fast retransmit?**
A: After 3 duplicate ACKs, retransmit immediately without waiting for timeout. Faster recovery since network is still partially working.

**19. Explain TCP termination.**
A: 4-way: FIN → ACK → FIN → ACK. Each direction closes independently (full-duplex). Initiator enters TIME_WAIT.

**20. Why does TIME_WAIT exist?**
A: 1) Ensure final ACK arrives (resend if peer retransmits FIN). 2) Let old duplicate packets expire before reusing the port pair.

**21. What happens if SYN is lost?**
A: Client retransmits SYN with exponential backoff (1s, 2s, 4s...). After max retries, connection fails.

**22. A packet is lost during transfer. What happens?**
A: Receiver sends dup ACKs. After 3, sender fast-retransmits. TCP reorders and delivers in sequence. Application is unaware.

**23. Why is UDP preferred for video streaming?**
A: Late retransmitted frames are useless — playback has moved on. Lower latency matters more than perfect delivery.

**24. Can you build reliability on UDP?**
A: Yes. QUIC (HTTP/3) does this — adds reliability, ordering, encryption over UDP with per-stream independence.

### DNS (25–30)

**25. What is DNS?**
A: Translates domain names to IPs. Hierarchical: Root → TLD → Authoritative. Uses UDP:53. Caches results with TTL.

**26. How does DNS resolution work?**
A: Browser/OS cache → recursive resolver → root (referral) → TLD (referral) → authoritative (answer). Each level cached.

**27. Recursive vs iterative DNS?**
A: Recursive = resolver returns final answer. Iterative = server gives referral. Client→resolver is recursive. Resolver→servers is iterative.

**28. Important DNS records?**
A: A (domain→IPv4), AAAA (→IPv6), CNAME (alias→domain), MX (mail server), NS (authoritative server), TXT (verification/SPF).

**29. Why does DNS use UDP?**
A: Queries are small (fit in 1 UDP datagram). No connection overhead. TCP used for zone transfers and large responses.

**30. DNS record changed but old IP still shows. Why?**
A: Caching. Old record cached at browser/OS/resolver with unexpired TTL. Must wait for TTL to expire.

### HTTP/HTTPS (31–42)

**31. What is HTTP?**
A: Application-layer protocol for web. Request (method + URL + headers + body) → Response (status + headers + body). Stateless.

**32. HTTP methods and idempotency?**
A: GET (read, idempotent), POST (create, NOT idempotent), PUT (replace, idempotent), PATCH (partial update), DELETE (idempotent).

**33. 401 vs 403?**
A: 401 = authentication needed ("who are you?"). 403 = authenticated but not authorized ("no permission").

**34. Why is HTTP stateless?**
A: Simplicity and scalability. Any server can handle any request. State maintained via cookies/sessions.

**35. How do cookies work?**
A: Server sends Set-Cookie → browser stores → browser sends Cookie header on every subsequent request → server identifies client.

**36. HTTP/1.1 vs HTTP/2?**
A: HTTP/2: binary framing, multiplexed streams over 1 TCP connection, HPACK header compression. Eliminates HTTP-level HOL blocking.

**37. What is HOL blocking?**
A: In HTTP/1.1: slow response blocks all others on that connection. HTTP/2 solves at HTTP level but TCP-level HOL remains. HTTP/3 solves both.

**38. HTTP vs HTTPS?**
A: HTTPS = HTTP + TLS. Encrypts data (confidentiality), prevents tampering (integrity), verifies server identity (authentication).

**39. What is TLS?**
A: Transport Layer Security. Handshake: exchange hellos, verify certificate, key exchange (DH) → derive symmetric session keys → encrypted communication.

**40. Symmetric vs asymmetric encryption?**
A: Symmetric = same key, fast, used for data. Asymmetric = public/private keys, slow, used for key exchange. TLS uses both (hybrid).

**41. What is a certificate?**
A: Binds domain + public key, signed by a CA. Browser verifies CA signature, domain match, expiry. Proves server identity.

**42. What is forward secrecy?**
A: Ephemeral DH keys per session. Compromising server's private key later can't decrypt past sessions. Mandatory in TLS 1.3.

### Routing/IP (43–46)

**43. What is NAT?**
A: Translates private IPs to public IP. PAT uses port numbers to multiplex. Allows many devices to share one public IP.

**44. What is longest prefix match?**
A: Router picks the most specific matching route. /24 beats /16 beats /8 beats default (0.0.0.0/0).

**45. What is TTL?**
A: Decremented at each router hop. At 0, packet dropped + ICMP Time Exceeded sent. Prevents infinite loops. Used by traceroute.

**46. Private vs public IP?**
A: Private (10.x, 172.16-31.x, 192.168.x) = internal, not internet-routable. Public = globally unique, routable. NAT bridges them.

### Real-World (47–50)

**47. What happens when you type https://google.com?**
A: URL parse → DNS lookup → ARP for gateway → TCP handshake → TLS handshake → HTTP request → server processing (CDN/LB) → HTTP response → browser renders.

**48. What is a CDN?**
A: Geographically distributed cache servers. Serves content from nearby edge → reduces latency. For static assets (images, CSS, JS).

**49. Forward proxy vs reverse proxy?**
A: Forward proxy = client side, hides client (filtering, anonymity). Reverse proxy = server side, hides servers (load balancing, SSL termination, caching).

**50. Ping works but HTTP doesn't. Why?**
A: Different protocols/ports. Firewall may allow ICMP but block TCP:80/443. Web server may not be running. TLS cert issue. Application error.

### Additional Topics (51–58)

**51. WebSocket vs SSE vs long polling?**
A: Long polling = repeated HTTP requests (high overhead). SSE = one-way server push over persistent HTTP (auto-reconnect). WebSocket = full-duplex bidirectional over persistent TCP (starts with HTTP upgrade). Use SSE for server-push, WebSocket for bidirectional.

**52. How does a WebSocket connection start?**
A: Client sends HTTP GET with `Upgrade: websocket` header. Server responds 101 Switching Protocols. Connection upgrades to persistent full-duplex WebSocket. No more HTTP framing.

**53. REST vs gRPC?**
A: REST = resource-oriented, JSON, HTTP, browser-friendly, public APIs. gRPC = action-oriented, protobuf (binary), HTTP/2, streaming, internal microservices. REST for external; gRPC for internal high-performance.

**54. What is Nagle's algorithm?**
A: Buffers small TCP writes until ACK arrives or MSS is full. Improves efficiency but adds latency. Disable with TCP_NODELAY for real-time apps. Interacts badly with Delayed ACK.

**55. MTU vs MSS?**
A: MTU = max IP packet size (1500 Ethernet). MSS = max TCP payload (1460 = MTU−40). MSS negotiated during handshake to avoid fragmentation.

**56. Distance vector vs link state routing?**
A: Distance vector (RIP) = share tables with neighbors, Bellman-Ford, slow convergence. Link state (OSPF) = flood topology, Dijkstra's, fast convergence. OSPF is superior and standard.

**57. CSMA/CD vs CSMA/CA?**
A: CSMA/CD = wired Ethernet, detect collisions during TX, obsolete with switches. CSMA/CA = Wi-Fi, avoid collisions (can't detect during TX). Wi-Fi uses backoff + RTS/CTS.

**58. What is classful addressing and why was it replaced?**
A: Original IPv4 scheme with fixed classes: A(/8), B(/16), C(/24). Extremely wasteful — 300 hosts needed a whole Class B (65K addresses). Replaced by CIDR which allows any prefix length.

---

## 6. Rapid-Fire Revision

**Q:** What port does HTTPS use?
**A:** 443.

**Q:** What port does DNS use?
**A:** 53 (UDP for queries, TCP for zone transfers).

**Q:** What port does SSH use?
**A:** 22.

**Q:** How many bits in an IPv4 address?
**A:** 32 bits.

**Q:** How many bits in a MAC address?
**A:** 48 bits.

**Q:** How many bits in a port number?
**A:** 16 bits (0–65535).

**Q:** What is the broadcast MAC address?
**A:** FF:FF:FF:FF:FF:FF.

**Q:** What is the loopback IP?
**A:** 127.0.0.1.

**Q:** What does a 169.254.x.x address mean?
**A:** DHCP failed. APIPA self-assigned address.

**Q:** TCP header size?
**A:** 20 bytes minimum (up to 60 with options).

**Q:** UDP header size?
**A:** 8 bytes.

**Q:** Ethernet MTU?
**A:** 1500 bytes.

**Q:** What is the 5-tuple?
**A:** (Source IP, Source Port, Destination IP, Destination Port, Protocol).

**Q:** What does ARP do?
**A:** Resolves IP address to MAC address on the local network.

**Q:** What does DNS do?
**A:** Resolves domain name to IP address.

**Q:** How many steps in the TCP handshake?
**A:** 3 (SYN → SYN-ACK → ACK).

**Q:** How many steps in TCP termination?
**A:** 4 (FIN → ACK → FIN → ACK).

**Q:** What is TIME_WAIT duration?
**A:** 2 × MSL (typically 120 seconds).

**Q:** Is GET idempotent?
**A:** Yes.

**Q:** Is POST idempotent?
**A:** No.

**Q:** What layer does a router operate at?
**A:** Layer 3 (Network).

**Q:** What layer does a switch operate at?
**A:** Layer 2 (Data Link).

**Q:** What protocol does ping use?
**A:** ICMP (Echo Request / Echo Reply).

**Q:** What protocol does traceroute exploit?
**A:** ICMP Time Exceeded (via incrementing TTL).

**Q:** TCP or UDP: which does HTTP use?
**A:** TCP (HTTP/1.1, HTTP/2). QUIC/UDP (HTTP/3).

**Q:** What does the ACK number in TCP mean?
**A:** The next byte the receiver expects.

**Q:** What triggers fast retransmit?
**A:** 3 duplicate ACKs.

**Q:** What happens to cwnd on timeout?
**A:** cwnd drops to 1 MSS, back to slow start.

**Q:** What happens to cwnd on 3 dup ACKs?
**A:** cwnd halved (fast recovery).

**Q:** What does a CA do?
**A:** Verifies domain ownership and signs digital certificates.

**Q:** TLS uses which encryption for bulk data?
**A:** Symmetric (AES or ChaCha20).

**Q:** TLS uses which encryption for key exchange?
**A:** Asymmetric (Diffie-Hellman).

**Q:** How many RTTs does TLS 1.3 handshake take?
**A:** 1 RTT (0-RTT for resumption).

**Q:** What's the 301 vs 302 difference?
**A:** 301 = permanent redirect (cached). 302 = temporary redirect.

**Q:** What's the 401 vs 403 difference?
**A:** 401 = not authenticated. 403 = authenticated but not authorized.

**Q:** What does a CNAME record do?
**A:** Aliases one domain to another domain.

**Q:** What does an MX record do?
**A:** Specifies the mail server for a domain.

**Q:** HTTP/2 key feature?
**A:** Multiplexed streams over a single TCP connection.

**Q:** HTTP/3 key feature?
**A:** QUIC over UDP — independent streams, no HOL blocking.

**Q:** What does NAT stand for?
**A:** Network Address Translation.

**Q:** Private IP ranges?
**A:** 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16.

**Q:** What does `rwnd` control?
**A:** Flow control — receiver's available buffer space.

**Q:** What does `cwnd` control?
**A:** Congestion control — sender's estimate of safe sending rate.

**Q:** What is slow start?
**A:** cwnd starts at 1 MSS, doubles every RTT (exponential) until ssthresh.

**Q:** What is congestion avoidance?
**A:** After ssthresh, cwnd grows by 1 MSS per RTT (linear).

**Q:** What function creates a new socket for each client on the server?
**A:** `accept()`.

**Q:** Forward proxy protects whom?
**A:** The client (hides client identity).

**Q:** Reverse proxy protects whom?
**A:** The server (hides server details, load balances).

**Q:** OSI has how many layers?
**A:** 7.

**Q:** TCP/IP has how many layers?
**A:** 4 (or 5).

**Q:** WebSocket starts with what HTTP status?
**A:** 101 Switching Protocols.

**Q:** Is SSE bidirectional?
**A:** No — server-to-client only.

**Q:** WebSocket is which direction?
**A:** Full-duplex bidirectional.

**Q:** gRPC uses what serialization format?
**A:** Protocol Buffers (protobuf) — binary.

**Q:** gRPC runs over what transport?
**A:** HTTP/2.

**Q:** REST is resource-oriented or action-oriented?
**A:** Resource-oriented (nouns: /users/123).

**Q:** What does Nagle's algorithm do?
**A:** Buffers small TCP writes until ACK arrives or MSS is full.

**Q:** How to disable Nagle?
**A:** Set TCP_NODELAY socket option.

**Q:** MSS for Ethernet?
**A:** 1460 bytes (MTU 1500 − 20 IP − 20 TCP).

**Q:** SMTP sends or retrieves email?
**A:** Sends (push). POP3/IMAP retrieve.

**Q:** POP3 vs IMAP in one sentence?
**A:** POP3 downloads and deletes. IMAP syncs and keeps on server.

**Q:** CSMA/CD is for wired or wireless?
**A:** Wired (old Ethernet). Wi-Fi uses CSMA/CA.

**Q:** TCP's retransmission is closest to which ARQ?
**A:** Selective Repeat.

**Q:** What did classful addressing use?
**A:** Fixed classes: A(/8), B(/16), C(/24). Replaced by CIDR.

**Q:** What is the count-to-infinity problem?
**A:** Distance vector routers keep incrementing hop count for a failed route through each other. Slow convergence.

**Q:** OSPF uses which algorithm?
**A:** Dijkstra's shortest path.

---

## 7. Final Interview Checklist

Before considering your CN preparation complete, you should be able to explain:

- [ ] OSI model (7 layers) and TCP/IP model (4 layers) — what each layer does
- [ ] Encapsulation and decapsulation — how data moves through the stack
- [ ] MAC vs IP vs Port — purpose and scope of each
- [ ] How a switch learns MAC addresses and forwards frames
- [ ] Hub vs Switch vs Router — layer, function, and domains
- [ ] Repeater vs Hub vs Bridge vs Switch vs Router vs Gateway — full hierarchy
- [ ] Collision domain vs broadcast domain
- [ ] CSMA/CD vs CSMA/CA — wired vs wireless medium access
- [ ] ARP — what it does, how it works, when it's used
- [ ] Same-LAN communication flow vs cross-network communication flow
- [ ] IPv4 addressing — subnet mask, network/host portions, private/public IPs
- [ ] Classful addressing — what it was and why CIDR replaced it
- [ ] Subnetting — calculate network address, broadcast address, usable hosts from CIDR
- [ ] How routing works — routing table, longest prefix match, default route
- [ ] Distance vector (RIP) vs link state (OSPF) — basic difference
- [ ] NAT/PAT — why it exists and how it works
- [ ] TTL, ICMP, ping, traceroute
- [ ] MTU vs MSS and why fragmentation is avoided
- [ ] **TCP 3-way handshake on a whiteboard** — SYN, SYN-ACK, ACK with sequence numbers
- [ ] TCP reliable delivery — sequence numbers, ACKs, retransmission, fast retransmit
- [ ] Stop-and-Wait vs Go-Back-N vs Selective Repeat — and how TCP relates
- [ ] **Flow control vs congestion control** — explain the difference clearly
- [ ] Slow start → congestion avoidance → fast recovery → timeout behavior
- [ ] **TCP 4-way termination** — FIN, ACK, FIN, ACK and why 4 steps
- [ ] TIME_WAIT — what it is and why it exists
- [ ] Nagle's algorithm and TCP_NODELAY — when and why to disable
- [ ] TCP vs UDP — when to use each
- [ ] DNS resolution — full flow from browser to authoritative server
- [ ] DNS records — A, AAAA, CNAME, MX, NS, TXT
- [ ] HTTP methods — GET, POST, PUT, PATCH, DELETE and idempotency
- [ ] HTTP status codes — 200, 201, 301, 302, 304, 400, 401, 403, 404, 500, 502, 503
- [ ] HTTP statelessness — how cookies and sessions maintain state
- [ ] HTTP/1.1 vs HTTP/2 vs HTTP/3 — multiplexing, HOL blocking, QUIC
- [ ] WebSocket vs SSE vs long polling — when to use each
- [ ] REST vs gRPC — when to choose each
- [ ] WebSocket handshake (HTTP upgrade → 101 → full-duplex)
- [ ] Symmetric vs asymmetric encryption and why TLS uses both
- [ ] TLS handshake — certificate verification, key exchange, session keys
- [ ] What a certificate and CA do
- [ ] **"What happens when you type https://google.com?"** — the complete end-to-end flow
- [ ] Proxy vs reverse proxy vs load balancer vs CDN
- [ ] Socket lifecycle — bind, listen, accept, connect
- [ ] SMTP, POP3, IMAP — email protocol roles
- [ ] Common troubleshooting scenarios (ping works but HTTP doesn't, etc.)
