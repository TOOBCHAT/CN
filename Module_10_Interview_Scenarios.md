# Module 10 — Interview Scenarios

> **No new theory. Pure application of everything from Modules 1–9.**
> Every question has the ideal answer immediately below it.

---

## TCP Scenarios

---

**Q: A TCP packet is lost. What happens?**

**Ideal Answer:**
"The receiver detects a gap in sequence numbers when later segments arrive. It sends duplicate ACKs for the last in-order byte it received. After the sender receives 3 duplicate ACKs, it performs fast retransmit — resending the lost segment immediately without waiting for the timeout. If no ACKs arrive at all (severe loss), the sender's retransmission timeout (RTO) expires and it retransmits, dropping cwnd to 1 MSS and re-entering slow start."

---

**Q: A packet arrives out of order. What happens?**

**Ideal Answer:**
"TCP at the receiver buffers the out-of-order segment and sends a duplicate ACK for the last in-order byte. When the missing segment arrives, TCP reorders everything and delivers the data to the application in the correct sequence. The application never sees out-of-order data — TCP handles reordering transparently."

---

**Q: Why does TCP retransmit?**

**Ideal Answer:**
"TCP retransmits because the underlying IP network is unreliable — packets can be lost due to congestion, router buffer overflow, or errors. TCP detects loss via two mechanisms: timeout (no ACK received within RTO) and 3 duplicate ACKs (receiver signals a gap). Retransmission ensures every byte is eventually delivered to the application."

---

**Q: What happens when congestion occurs?**

**Ideal Answer:**
"When the network is congested, routers drop packets due to full buffers. TCP detects this loss. If detected via 3 duplicate ACKs (mild congestion), TCP halves its congestion window (cwnd) and enters fast recovery — this is the 'multiplicative decrease' part of AIMD. If detected via timeout (severe congestion), cwnd drops to 1 MSS and TCP restarts slow start. Both responses reduce the sending rate to alleviate congestion."

---

**Q: What happens if SYN is lost?**

**Ideal Answer:**
"The client never receives a SYN-ACK. After the retransmission timeout (typically starting at 1 second), the client retransmits the SYN. It retries with exponential backoff — 1s, 2s, 4s, 8s, etc. After a maximum number of retries (typically 5-6), the connection attempt fails and the application gets a 'connection timed out' error."

---

**Q: What happens if SYN-ACK is lost?**

**Ideal Answer:**
"The client doesn't receive the SYN-ACK, so from the client's perspective it looks the same as a lost SYN — the client retransmits its SYN. Meanwhile, the server doesn't receive the final ACK, so it retransmits its SYN-ACK. Eventually they synchronize and the handshake completes, or both sides time out."

---

**Q: What happens if the final ACK of the handshake is lost?**

**Ideal Answer:**
"The server stays in SYN_RECEIVED state and will retransmit its SYN-ACK. The client is already in ESTABLISHED state. If the client starts sending data, those segments will carry the ACK flag and the correct acknowledgment number, which effectively completes the handshake for the server. So in practice, the connection still establishes."

---

**Q: Why does TIME_WAIT exist?**

**Ideal Answer:**
"Two reasons. First, to ensure the final ACK reaches the other side. If it's lost, the peer will retransmit its FIN, and the host in TIME_WAIT can resend the ACK. Second, to allow old duplicate packets from the closed connection to expire before a new connection reuses the same port pair. Without TIME_WAIT, a stale packet from an old connection could be misinterpreted as belonging to a new connection. TIME_WAIT lasts 2×MSL (typically 120 seconds)."

---

**Q: What happens if a FIN is lost during connection termination?**

**Ideal Answer:**
"If the FIN is lost, the receiver never knows the sender wants to close. The sender doesn't receive an ACK for its FIN, so it retransmits the FIN after a timeout. This continues with exponential backoff until either the FIN gets through and the termination proceeds, or the retransmission limit is reached and the connection is reset."

---

**Q: Can TCP guarantee zero packet loss?**

**Ideal Answer:**
"TCP guarantees delivery, not that packets won't be lost in the network. Packets can and do get lost — TCP detects this and retransmits. What TCP guarantees is that the application will eventually receive all data in order, or the connection will be terminated with an error. The network is unreliable; TCP provides reliability on top of it."

---

## HTTP Scenarios

---

**Q: Why is HTTP stateless?**

**Ideal Answer:**
"HTTP is stateless by design for simplicity and scalability. Each request is independent — the server doesn't need to remember previous requests. This means any server behind a load balancer can handle any request without needing shared state. It also makes the protocol simpler and more robust — if a server crashes, no client state is lost. State is maintained externally using cookies and sessions when needed."

---

**Q: How do cookies maintain state over a stateless protocol?**

**Ideal Answer:**
"The server sends a Set-Cookie header in the response. The browser stores the cookie and automatically includes it in every subsequent request to that domain via the Cookie header. The server reads the cookie to identify the client or look up their session. The protocol itself remains stateless — each request is still independent — but the cookie provides the context to link requests to a specific user or session."

---

**Q: An HTTP request is slow — where could the problem be?**

**Ideal Answer:**
"Multiple possible bottlenecks: DNS resolution could be slow (uncached, distant DNS server). TCP handshake adds latency (especially over high-RTT links). TLS handshake adds 1-2 more RTTs. The server could be slow processing the request (heavy database queries, CPU load). The response could be large and bandwidth-limited. Network congestion could cause packet loss and retransmission. The connection might not be reusing keep-alive, so every request pays the full TCP+TLS cost. No CDN means content is served from a distant origin. I'd troubleshoot by measuring time spent in each phase — DNS, connection, TLS, time-to-first-byte, content download."

---

**Q: What happens if the server returns 500?**

**Ideal Answer:**
"A 500 Internal Server Error means the server encountered an unexpected error while processing the request — typically an unhandled exception, application crash, or misconfiguration. It's a server-side problem, not the client's fault. The client can retry the request, but if the bug is persistent, it will keep failing. The server admin needs to check error logs to diagnose and fix the issue."

---

**Q: HTTP vs HTTPS — what's the actual difference at the protocol level?**

**Ideal Answer:**
"The HTTP messages are identical. The difference is that HTTPS adds a TLS layer between HTTP and TCP. After the TCP handshake, a TLS handshake occurs — the server's identity is verified via its certificate, symmetric session keys are derived, and all subsequent HTTP data is encrypted and integrity-checked. The HTTP request and response themselves look the same — TLS is transparent to the HTTP layer."

---

**Q: Why was HTTP/2 created? What problem does it solve?**

**Ideal Answer:**
"HTTP/2 solves HTTP/1.1's head-of-line blocking and connection inefficiency. In HTTP/1.1, responses must come in order on each connection, and browsers open 6 parallel connections to work around this. HTTP/2 introduces multiplexing — multiple requests and responses flow concurrently over a single TCP connection as independent streams. It also adds binary framing (more efficient than text) and header compression (reduces repeated header overhead)."

---

**Q: How does HTTP/3 solve head-of-line blocking that HTTP/2 couldn't?**

**Ideal Answer:**
"HTTP/2 solved HTTP-level HOL blocking with multiplexed streams, but it still uses TCP, which delivers all bytes in order. If one TCP segment is lost, ALL streams are blocked until it's retransmitted. HTTP/3 uses QUIC over UDP, which handles each stream independently at the transport level. A lost packet for stream A only blocks stream A — streams B and C continue unaffected. This eliminates both HTTP-level and transport-level HOL blocking."

---

## DNS Scenarios

---

**Q: DNS server is unavailable. What happens?**

**Ideal Answer:**
"Domain names can't be resolved to IP addresses. Websites won't load when accessed by name, though cached DNS entries still work until their TTL expires. The OS typically has multiple DNS servers configured and will try alternates. You can still access servers by IP address directly. Tools like ping would fail with 'cannot resolve hostname' but succeed with an IP address."

---

**Q: DNS lookup is slow. What could be the reasons?**

**Ideal Answer:**
"The DNS result might not be cached anywhere — requiring the full recursive resolution (root → TLD → authoritative). The recursive resolver might be geographically distant. The authoritative DNS server might be slow or overloaded. Network congestion between the resolver and DNS servers. Using a slow ISP resolver instead of a fast public one like 8.8.8.8 or 1.1.1.1. Very low TTLs causing frequent re-resolution."

---

**Q: Why does DNS use caching?**

**Ideal Answer:**
"Without caching, every DNS lookup would require traversing the full hierarchy — root, TLD, authoritative servers. That's 3+ network round trips per lookup, plus massive load on DNS infrastructure. Caching at every level (browser, OS, resolver) means most lookups are answered instantly from cache. This dramatically reduces latency for users and load on DNS servers."

---

**Q: What is the purpose of DNS TTL?**

**Ideal Answer:**
"TTL (Time to Live) specifies how long a DNS record can be cached before it must be refreshed from the authoritative source. It balances freshness vs performance: high TTL = fewer queries but slower propagation of changes. Low TTL = more queries but faster propagation. During a planned DNS migration, you'd lower the TTL in advance so the old cached records expire quickly when you make the change."

---

**Q: You changed a DNS record but users still see the old IP. Why?**

**Ideal Answer:**
"DNS caching. The old record is cached at multiple levels — browser, OS, recursive resolver — and the TTL hasn't expired yet. Users will see the old IP until all cached copies expire. This is why you should lower the TTL well before making changes. Some resolvers also ignore TTLs and cache longer. Browser caches might also need to be cleared manually."

---

## Routing Scenarios

---

**Q: Two computers are on different networks. How do they communicate?**

**Ideal Answer:**
"Computer A compares the destination IP with its own subnet mask and determines it's on a different network. A sends the packet to its default gateway (router). The router reads the destination IP, consults its routing table using longest prefix match, and forwards the packet to the next hop. This continues router by router until the packet reaches the destination's local network, where ARP resolves the final hop to the destination's MAC. The IP addresses stay the same end-to-end; the MAC addresses change at every hop."

---

**Q: How does a router determine where to forward a packet?**

**Ideal Answer:**
"The router extracts the destination IP from the IP header and searches its routing table. It uses longest prefix match — the most specific matching route wins. For example, a /24 route is preferred over a /16. The matching entry specifies the next hop IP and outgoing interface. If no route matches and there's no default route (0.0.0.0/0), the router drops the packet and sends ICMP Destination Unreachable."

---

**Q: What happens if there is no route for a destination?**

**Ideal Answer:**
"If no route matches and there's no default route, the router drops the packet and sends an ICMP Destination Unreachable (Network Unreachable) message back to the sender. The sender's application gets an error like 'No route to host.' In well-configured networks, a default route (0.0.0.0/0) acts as a catch-all to prevent this."

---

**Q: What does TTL do? What happens when it reaches 0?**

**Ideal Answer:**
"TTL is decremented by 1 at each router hop. When it reaches 0, the router drops the packet and sends an ICMP Time Exceeded message back to the sender. TTL prevents packets from looping forever in the network due to routing errors. Traceroute exploits this — it sends packets with TTL 1, 2, 3... and each router that drops a packet reveals its IP in the ICMP response."

---

**Q: What is the difference between ARP and routing?**

**Ideal Answer:**
"ARP operates at Layer 2 — it resolves IP addresses to MAC addresses on the local network. Routing operates at Layer 3 — it determines which network path a packet should take to reach a remote destination. ARP is used for the last mile (or first mile) of delivery — getting a frame to the next hop on the local link. Routing determines which next hop to use. They work together: the router makes a routing decision (Layer 3), then uses ARP (Layer 2) to get the next hop's MAC address."

---

## General / Troubleshooting Scenarios

---

**Q: Ping works but HTTP doesn't. Why?**

**Ideal Answer:**
"Ping uses ICMP, HTTP uses TCP port 80/443 — they're different protocols on different ports. Possible causes: a firewall is blocking TCP port 80/443 but allowing ICMP. The web server (Nginx/Apache) isn't running or is misconfigured. The server is listening on a different port. DNS resolves correctly but the server isn't serving HTTP. The server's application is crashing on requests. TCP connection may succeed but TLS handshake fails (certificate issue)."

---

**Q: DNS works but the website doesn't load. What could be wrong?**

**Ideal Answer:**
"DNS returned an IP, so name resolution works. Possible issues: the server at that IP is down. A firewall is blocking the connection (port 80/443). The TCP connection is refused (no service listening on that port). TLS certificate is expired or invalid (HTTPS error). The server returns an HTTP error (500, 502, 503). The DNS returned a wrong/stale IP. Network issues between you and the server (routing problem, congestion)."

---

**Q: Website is slow. How would you troubleshoot it?**

**Ideal Answer:**
"I'd break the request into phases and measure each one. DNS resolution time — if slow, try a different DNS resolver or check TTL. TCP connection time — indicates network latency. TLS handshake time — adds 1-2 RTTs. Time To First Byte (TTFB) — if high, the server is slow processing the request (check server logs, database queries). Content download time — if slow, bandwidth is limited or the response is too large (compress, optimize). I'd use browser DevTools Network tab to measure each phase, traceroute for path analysis, and check if a CDN is in use."

---

**Q: TCP connection succeeds but HTTP request fails. What could be happening?**

**Ideal Answer:**
"The TCP layer is working (handshake completed). Possible issues: TLS handshake failure — wrong TLS version, expired certificate, domain mismatch, unsupported cipher suite. The server returns an HTTP error — 400 (bad request format), 401/403 (auth issue), 500 (server crash). The request is blocked by a WAF (Web Application Firewall). Wrong Host header — virtual hosting requires the correct Host header to route to the right site. Server timeout — the application takes too long to respond."

---

**Q: You can access a website from one network but not another. Why?**

**Ideal Answer:**
"Different networks have different configurations that could cause this. The blocking network might have a firewall or ACL that blocks the destination. Different DNS resolvers might return different or stale IPs. The destination server might block certain IP ranges (geo-blocking, IP blacklisting). One network might require a proxy. There could be routing differences — one network has a valid path, the other doesn't. The ISP might do content filtering."

---

**Q: A user reports that a website loads on Wi-Fi but not on mobile data. What could cause this?**

**Ideal Answer:**
"The mobile carrier and Wi-Fi use different network paths. The carrier might use a different DNS server that returns a different or stale IP. The carrier might block the domain or port. The site might block the carrier's IP range. The carrier might use a transparent proxy that interferes with HTTPS. Mobile networks can have different MTU settings that cause fragmentation issues. IPv4/IPv6 differences — Wi-Fi might use IPv4 where the site works, mobile might prefer IPv6 where it doesn't (or vice versa)."

---

**Q: You deploy a new server but clients can't connect. TCP connection times out. What do you check?**

**Ideal Answer:**
"First, verify the server is actually running and listening on the expected port (use `netstat` or `ss`). Check if a firewall (OS-level or network-level) is blocking the port. Verify the server's security group / ACL rules (in cloud environments). Check DNS — does the domain resolve to the correct new IP? Verify network routing — is the server reachable (try ping, traceroute)? Check if the server's subnet and gateway are configured correctly. If behind a load balancer, verify the health check is passing."

---

**Q: Two microservices can ping each other but can't communicate over HTTP. What's wrong?**

**Ideal Answer:**
"Ping (ICMP) works, so L3 connectivity exists. The issue is at L4/L7. Check if the target service is listening on the expected port. Check firewall rules — the port might be blocked even though ICMP is allowed. Check if TLS is required but not configured. Verify the service is healthy (not crashing on startup). Check service discovery / DNS — the service name might resolve to the wrong IP. Check network policies (in Kubernetes, network policies can block specific ports while allowing ICMP)."

---

**Q: A file upload works for small files but fails for large files. Why?**

**Ideal Answer:**
"Several possibilities. The web server has a maximum request body size limit (Nginx default is 1MB — `client_max_body_size`). A reverse proxy or load balancer has a request size limit. The connection times out for large uploads on slow networks. The application has a file size limit. TCP window scaling might not be negotiated properly, limiting throughput. A WAF might block large payloads."

---

**Q: Users in India experience slow loading but users in the US do not. Why?**

**Ideal Answer:**
"Latency — the server is likely hosted in the US. Indian users have higher RTT, which means every TCP handshake, TLS handshake, and HTTP round-trip is slower. The solution is to use a CDN with edge servers in India to serve static content locally, and potentially deploy application servers in an Indian region. Also check if DNS resolution is slow for Indian users (using a distant DNS resolver)."

---

**Q: HTTPS works in Chrome but not in curl. Why?**

**Ideal Answer:**
"Most likely a TLS certificate issue. Chrome might have the necessary root CA certificates while curl's certificate bundle doesn't include it (especially common with newer CAs or custom/internal CAs). The server might require SNI (Server Name Indication), and an older curl version might not send it. The server might require a specific TLS version that curl doesn't negotiate by default. Use `curl -v` to see the exact TLS error."
