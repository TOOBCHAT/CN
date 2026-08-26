# Module 1 — Networking Fundamentals

---

## 1. What Is a Computer Network? `MUST KNOW`

A computer network is a set of devices (computers, servers, routers, switches) connected together to share data and resources.

**Why it exists:** Machines need to communicate — share files, access services (web, email, databases), distribute workloads.

**Interview angle:** You won't be asked "define a network," but you need this mental model: every networking concept exists to solve the problem of *reliably moving data from point A to point B across interconnected devices*.

---

## 2. Network Architecture `MUST KNOW`

### Client-Server Model
- **Client** initiates requests (browser, mobile app).
- **Server** listens, processes, and responds.
- Most of the internet works this way (HTTP, DNS, email).

### Peer-to-Peer (P2P)
- Every node is both client and server.
- Examples: BitTorrent, some blockchain nodes.
- Interview relevance: low — mentioned for completeness.

**Interview point:** When they ask "how does the web work?" — the answer is always client-server.

---

## 3. OSI Model `MUST KNOW`

The OSI model is a **conceptual framework** that standardizes how networking works into **7 layers**. Each layer has a specific job and communicates with the layers directly above and below it.

**Why it exists:** Without a layered model, changing one part of networking (e.g., switching from Wi-Fi to Ethernet) would break everything above it. Layers provide **abstraction** — each layer only cares about its own job.

| Layer | Name | What It Does | Key Protocols/Concepts | Data Unit |
|-------|------|-------------|----------------------|-----------|
| 7 | Application | User-facing services & protocols | HTTP, DNS, FTP, SMTP | Data |
| 6 | Presentation | Data formatting, encryption, compression | TLS/SSL, JPEG, ASCII | Data |
| 5 | Session | Manages sessions/connections | NetBIOS, RPC | Data |
| 4 | Transport | End-to-end delivery, reliability, flow control | TCP, UDP | **Segment** (TCP) / **Datagram** (UDP) |
| 3 | Network | Logical addressing & routing | IP, ICMP, ARP* | **Packet** |
| 2 | Data Link | Physical addressing & local delivery | Ethernet, MAC | **Frame** |
| 1 | Physical | Bits on the wire/air | Cables, Wi-Fi, signals | **Bits** |

> [!NOTE]
> In interviews, layers 5, 6, 7 are often grouped together as "Application layer" (which is what TCP/IP does). Don't spend time defending the distinction between Session and Presentation layers — interviewers rarely care.

**Mnemonic (top-down):** **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing

---

## 4. TCP/IP Model `MUST KNOW`

The TCP/IP model is the **practical model** that the internet actually uses. It has **4 layers** (sometimes shown as 5).

| TCP/IP Layer | Corresponds to OSI | Key Protocols |
|---|---|---|
| Application | OSI 5, 6, 7 | HTTP, DNS, FTP, SMTP, TLS |
| Transport | OSI 4 | TCP, UDP |
| Internet (Network) | OSI 3 | IP, ICMP, ARP |
| Network Access (Link) | OSI 1, 2 | Ethernet, Wi-Fi, MAC |

### OSI vs TCP/IP — What Interviewers Want to Hear

| Aspect | OSI | TCP/IP |
|--------|-----|--------|
| Layers | 7 | 4 (or 5) |
| Nature | Theoretical/reference model | Practical/implementation model |
| Development | ISO standard | Developed alongside the internet |
| Usage | Used to *explain* networking | Used to *build* networking |
| Session/Presentation | Separate layers | Merged into Application |

**Interview answer:** "OSI is a theoretical reference model with 7 layers. TCP/IP is the practical model the internet actually runs on, with 4 layers. OSI is useful for explaining concepts; TCP/IP is what's implemented in real systems. The main difference is that OSI separates Session, Presentation, and Application into 3 layers, while TCP/IP combines them into one Application layer."

---

## 5. What Each Layer Actually Does `MUST KNOW`

### Application Layer (L7)
- Where user-facing protocols live.
- HTTP for web, DNS for name resolution, SMTP for email.
- Your application code interacts at this layer (e.g., making HTTP requests).

### Transport Layer (L4)
- **End-to-end communication** between processes on different machines.
- TCP provides **reliable, ordered delivery** with flow/congestion control.
- UDP provides **fast, unreliable delivery** with no guarantees.
- Uses **port numbers** to identify which process gets the data.

### Network Layer (L3)
- **Logical addressing** (IP addresses) and **routing**.
- Determines the path data takes across networks.
- Routers operate at this layer.

### Data Link Layer (L2)
- **Physical addressing** (MAC addresses) and **local delivery** within a single network segment.
- Switches operate at this layer.
- Handles framing, error detection (CRC).

### Physical Layer (L1)
- Raw bits → electrical/optical/radio signals.
- Cables, connectors, frequencies.
- Interview relevance: very low — just know it exists.

---

## 6. Encapsulation & Decapsulation `MUST KNOW`

### Encapsulation (Sending Side)
As data moves **down** the stack, each layer wraps the data with its own header (and sometimes trailer).

```
Application:  [DATA]
Transport:    [TCP Header | DATA]                                        → Segment
Network:      [IP Header | TCP Header | DATA]                            → Packet
Data Link:    [ETH Header | IP Header | TCP Header | DATA | ETH Trailer] → Frame
Physical:     → Bits on the wire
```

**Each layer treats everything from the layer above as its payload.** The Transport layer doesn't know or care what's inside the application data. The Network layer treats the entire segment (TCP header + data) as its payload.

### Decapsulation (Receiving Side)
As data moves **up** the stack, each layer strips its header, reads the relevant information, and passes the payload up.

**Why this matters in interviews:** It shows you understand that networking is layered and each protocol only processes its own header. When an interviewer asks "what happens when a packet arrives at a host?" — you describe decapsulation: the NIC processes the frame header (checks MAC), the IP layer processes the IP header (checks destination IP), the transport layer processes the TCP/UDP header (routes to the correct port/process).

---

## 7. Data Unit Names `MUST KNOW`

| Layer | Data Unit | When to Use |
|-------|-----------|-------------|
| Transport (TCP) | **Segment** | Talking about TCP data |
| Transport (UDP) | **Datagram** | Talking about UDP data |
| Network | **Packet** | Talking about IP-level data |
| Data Link | **Frame** | Talking about Ethernet/local delivery |

**Interview tip:** Using the correct term shows precision. Don't call everything a "packet." Say "TCP segment" when discussing TCP, "frame" when discussing Ethernet.

> "Packet" is sometimes used loosely as a generic term — that's okay in casual conversation, but in an interview, precision matters.

---

## 8. Addressing: MAC vs IP vs Port `MUST KNOW`

### MAC Address (Layer 2)
- **48-bit** hardware address, burned into the NIC (network interface card).
- Format: `AA:BB:CC:DD:EE:FF` (6 octets in hex).
- **Scope: local network only.** Used to deliver frames within a single LAN/broadcast domain.
- Changes at every hop (routers rewrite the MAC addresses in the frame).
- Unique globally (in theory — manufacturers assign them).

### IP Address (Layer 3)
- **32-bit** (IPv4) or **128-bit** (IPv6) logical address.
- Format (IPv4): `192.168.1.10`
- **Scope: end-to-end.** Used to route packets across networks.
- Does NOT change as the packet traverses routers (except in NAT).
- Assigned by network configuration (DHCP or manual).

### Port Number (Layer 4)
- **16-bit** number (0–65535).
- **Scope: identifies a specific process/service** on a machine.
- A single IP address can have thousands of simultaneous connections, differentiated by ports.
- Well-known ports: HTTP = 80, HTTPS = 443, DNS = 53, SSH = 22.

### The Key Analogy

| Address Type | Real-World Analogy | Purpose |
|---|---|---|
| MAC Address | Name on the mailbox at this building | Local delivery to the right device |
| IP Address | Street address + city + country | End-to-end routing across the world |
| Port Number | Apartment number in the building | Deliver to the right application/process |

### Critical Interview Distinction

> **MAC address identifies a device on a local network.**
> **IP address identifies a device across the internet.**
> **Port number identifies a process on a device.**

A complete network conversation is identified by: `(Source IP, Source Port, Destination IP, Destination Port, Protocol)` — this is called a **5-tuple**.

**How they work together:** When you send a request to `google.com:443`:
1. IP address (`142.250.x.x`) routes the packet across the internet to Google's server.
2. Port `443` tells the server to hand the data to the HTTPS process.
3. At each hop, the MAC address changes — it always points to the *next device* on the local link (your router, then the next router, etc.).

---

## 9. LAN and WAN `GOOD TO KNOW`

### LAN (Local Area Network)
- A network within a small geographic area (office, home, building).
- Devices communicate using **MAC addresses** and **switches**.
- Typically Ethernet or Wi-Fi.
- High speed (1 Gbps+), low latency.

### WAN (Wide Area Network)
- Spans large geographic areas (cities, countries).
- The internet is the largest WAN.
- Connects multiple LANs via **routers**.
- Lower speed, higher latency compared to LAN.

**Interview point:** "A LAN connects devices in the same local network using switches. A WAN connects different networks across large distances using routers. The internet is a WAN connecting millions of LANs."

---

## 10. Bandwidth, Throughput, Latency `MUST KNOW`

### Bandwidth
- **Maximum** data transfer rate of a link (theoretical capacity).
- Measured in bits per second (bps, Mbps, Gbps).
- Think of it as the width of the pipe.

### Throughput
- **Actual** data transfer rate achieved.
- Always ≤ bandwidth (due to congestion, protocol overhead, packet loss, etc.).
- Think of it as how much water actually flows through the pipe.

### Latency
- **Time delay** for data to travel from source to destination.
- Measured in milliseconds (ms).
- Components: propagation delay + transmission delay + queuing delay + processing delay.
- **RTT (Round-Trip Time):** Time for a packet to go to the destination and back. Commonly used in TCP discussions.

**Interview distinction:**
> Bandwidth = max capacity (100 Mbps link)
> Throughput = actual performance (80 Mbps achieved)
> Latency = delay (20ms one way, 40ms RTT)

**Example:** "A 100 Mbps Ethernet link (bandwidth) might achieve only 70 Mbps throughput due to protocol overhead and congestion, with 5ms latency."

---

## 11. Unicast, Broadcast, Multicast `GOOD TO KNOW`

| Type | Description | Example |
|------|-------------|---------|
| **Unicast** | One-to-one communication | HTTP request from client to server |
| **Broadcast** | One-to-all on the local network | ARP request ("who has 192.168.1.1?") |
| **Multicast** | One-to-many (specific group) | Video streaming to subscribers |

**Interview point:** ARP uses broadcast. Most internet traffic is unicast. Broadcast doesn't cross routers (stays within the broadcast domain).

---

## 12. Circuit Switching vs Packet Switching `GOOD TO KNOW`

### Circuit Switching
- A **dedicated communication path** is established before data transfer.
- The path remains reserved for the entire session.
- Example: traditional telephone networks (PSTN).
- **Pros:** Guaranteed bandwidth, constant delay.
- **Cons:** Wasteful — the circuit is reserved even during silence.

### Packet Switching
- Data is broken into **packets** that are sent independently.
- Each packet can take a different path.
- Packets are reassembled at the destination.
- Example: **the internet.**
- **Pros:** Efficient use of bandwidth, shared resources.
- **Cons:** Variable delay, packets can arrive out of order.

**Why the internet uses packet switching:** It's more efficient — no wasted bandwidth during idle periods, and the network can serve many users simultaneously.

**Interview answer:** "The internet uses packet switching because it's more efficient than circuit switching. In circuit switching, a dedicated path is reserved for the entire session, wasting bandwidth during idle periods. In packet switching, data is broken into packets that share network resources, allowing many users to use the same links. The tradeoff is that packets can arrive out of order or be lost, which is why TCP exists — to provide reliability over an unreliable packet-switched network."

---

## 13. How Data Moves Through the Networking Stack `MUST KNOW`

This ties everything together. When your browser requests `http://example.com`:

### Sending Side (Your Computer)

1. **Application Layer:** The browser creates an HTTP GET request.
2. **Transport Layer:** TCP wraps the HTTP data in a segment. Adds source port (random high port, e.g., 52431) and destination port (80). Manages connection state (3-way handshake already done).
3. **Network Layer:** IP wraps the segment in a packet. Adds source IP (your IP, e.g., 192.168.1.10) and destination IP (resolved via DNS, e.g., 93.184.216.34).
4. **Data Link Layer:** Ethernet wraps the packet in a frame. Adds source MAC (your NIC's MAC) and destination MAC (**the router/default gateway's MAC** — NOT the server's MAC).
5. **Physical Layer:** Frame is converted to electrical/optical signals and sent on the wire.

### At Each Router (Hop)

1. Router receives the frame → strips the Ethernet header (decapsulation to L2).
2. Reads the IP header → looks at destination IP → consults routing table → determines next hop.
3. Creates a **new frame** with the next hop's MAC address as the destination MAC.
4. Sends the new frame out the appropriate interface.

> [!IMPORTANT]
> The IP addresses (source and destination) stay the same throughout the journey (ignoring NAT). The MAC addresses change at every hop — they always refer to the current and next device on the local link.

### Receiving Side (The Server)

1. **Data Link:** Checks destination MAC — it matches → accepts the frame → strips Ethernet header.
2. **Network:** Checks destination IP — it matches → strips IP header.
3. **Transport:** Checks destination port (80) → routes to the HTTP server process → strips TCP header.
4. **Application:** HTTP server reads the HTTP request and processes it.

---

## Interview Questions + Answers

### Must-Know Questions

---

**Q1: Explain the OSI model and its layers.**

**Ideal Answer:**
"The OSI model is a 7-layer reference model that standardizes how network communication works. From bottom to top: Physical handles raw bits on the wire. Data Link handles local delivery using MAC addresses. Network handles routing using IP addresses. Transport provides end-to-end delivery using TCP or UDP with port numbers. Session, Presentation, and Application handle session management, data formatting, and user-facing protocols respectively. In practice, the TCP/IP model combines the top 3 into one Application layer, which is what the internet actually uses."

---

**Q2: What is the difference between OSI and TCP/IP?**

**Ideal Answer:**
"OSI is a 7-layer theoretical model; TCP/IP is a 4-layer practical model that the internet uses. The main difference is that OSI has separate Session, Presentation, and Application layers, while TCP/IP merges them into one Application layer. OSI was designed as a reference framework; TCP/IP was developed alongside the actual internet protocols."

---

**Q3: What is encapsulation?**

**Ideal Answer:**
"Encapsulation is the process where each networking layer adds its own header (and sometimes trailer) to the data from the layer above. The upper layer's entire output becomes the payload for the lower layer. For example, the Transport layer takes application data and adds a TCP header to create a segment. The Network layer takes that segment and adds an IP header to create a packet. The Data Link layer takes the packet and adds an Ethernet header and trailer to create a frame. On the receiving side, decapsulation happens — each layer strips its header and passes the payload up."

---

**Q4: What is the difference between a packet, segment, frame, and datagram?**

**Ideal Answer:**
"These are the data units at different layers. A segment is the Transport layer unit for TCP. A datagram is the Transport layer unit for UDP. A packet is the Network layer unit (IP). A frame is the Data Link layer unit (Ethernet). Each one wraps the layer above — a frame contains a packet, which contains a segment or datagram, which contains the application data."

---

**Q5: What is the difference between MAC address, IP address, and port number?**

**Ideal Answer:**
"A MAC address is a 48-bit hardware address used for local delivery within a LAN — it identifies a device on the local network segment and changes at every router hop. An IP address is a 32-bit (IPv4) or 128-bit (IPv6) logical address used for end-to-end routing across networks — it stays the same throughout the journey. A port number is a 16-bit number that identifies a specific process or service on a machine, allowing multiple applications to use the network simultaneously. Together, they work in a hierarchy: IP routes the packet to the right machine, port delivers to the right process, and MAC handles local link delivery."

> **Follow-up Q: Why do we need both MAC and IP addresses?**
>
> **Ideal Answer:**
> "IP addresses handle routing across networks — they're logical and can change. MAC addresses handle local delivery on a single network segment — they're tied to hardware. A router uses the destination IP to decide where to forward a packet, then uses MAC addressing (via ARP) to actually deliver the frame to the next hop on the local link. You can't route globally with MAC addresses because they have no hierarchical structure — routers would need a table entry for every device in the world. IP addresses are hierarchical (network + host portions), making routing scalable."

---

**Q6: What is the difference between bandwidth, throughput, and latency?**

**Ideal Answer:**
"Bandwidth is the maximum theoretical data rate of a link — like the width of a pipe. Throughput is the actual data rate achieved in practice — always equal to or less than bandwidth due to congestion, overhead, and other factors. Latency is the time delay for data to travel from source to destination. For example, a 1 Gbps link (bandwidth) might achieve 700 Mbps throughput with 10ms latency."

---

**Q7: What is the difference between circuit switching and packet switching?**

**Ideal Answer:**
"In circuit switching, a dedicated path is established and reserved for the entire duration of communication — like a phone call. In packet switching, data is split into packets that are sent independently and may take different paths — like the internet. The internet uses packet switching because it's more efficient: bandwidth isn't wasted during idle periods and many users can share the same links. The tradeoff is packets can arrive out of order or be lost, which is why reliability protocols like TCP exist."

---

**Q8: How does data move through the networking stack?**

**Ideal Answer:**
"On the sending side, the application creates data, the transport layer adds TCP/UDP headers with port numbers to create a segment, the network layer adds IP headers with source/destination IPs to create a packet, and the data link layer adds MAC headers with source/destination MACs to create a frame. The frame is sent as bits on the wire. At each router, the frame is stripped, the IP header is read for routing decisions, and a new frame is created with the next hop's MAC address. On the receiving side, each layer strips its header and passes the payload up until the application receives the data. The key insight is that MAC addresses change at every hop, but IP addresses remain the same end-to-end."

---

**Q9: What is RTT and why is it important?**

**Ideal Answer:**
"RTT — Round-Trip Time — is the time for a packet to travel from source to destination and back. It's critical in networking because many protocols depend on it. TCP uses RTT to set retransmission timeouts — if an ACK doesn't come back within the expected RTT, TCP retransmits. High RTT directly impacts application performance — for example, a TCP handshake takes 1 RTT, and TLS adds another 1–2 RTTs, so high latency means slow connection setup."

---

**Q10: Explain unicast, broadcast, and multicast.**

**Ideal Answer:**
"Unicast is one-to-one communication — a specific sender sends to a specific receiver, like an HTTP request. Broadcast is one-to-all within a network segment — the sender sends to every device on the local network, like an ARP request. Multicast is one-to-many — the sender sends to a specific group of interested receivers, like video streaming. Broadcast traffic doesn't cross routers, which is why broadcast domains are bounded by routers."

---

### Common Mistakes

| Mistake | Correction |
|---------|------------|
| "MAC addresses are used for routing across the internet" | MAC addresses are **only** for local delivery. IP addresses are used for routing across networks. MAC changes at every hop. |
| "Packets and frames are the same thing" | A frame is a Layer 2 unit (contains MAC headers). A packet is a Layer 3 unit (contains IP headers). A frame *wraps* a packet. |
| "IP address changes at every hop" | IP address stays the same end-to-end (except with NAT). It's the MAC address that changes at every hop. |
| "OSI model is what the internet uses" | The internet uses the TCP/IP model. OSI is a reference/teaching model. |
| "Bandwidth and throughput are the same" | Bandwidth = theoretical max capacity. Throughput = actual achieved rate ≤ bandwidth. |
| "TCP operates at the network layer" | TCP operates at the **Transport layer** (L4). IP operates at the Network layer (L3). |

---

### Interview Takeaways — Module 1

1. **OSI has 7 layers; TCP/IP has 4.** The internet uses TCP/IP. OSI is the reference model used for teaching.
2. **Encapsulation:** Each layer adds its own header. Upper layer's output = lower layer's payload.
3. **Data units:** Frame (L2) → Packet (L3) → Segment/Datagram (L4) → Data (L7).
4. **MAC = local delivery (changes at every hop). IP = end-to-end routing (stays same). Port = identifies process.**
5. **5-tuple** `(src IP, src port, dst IP, dst port, protocol)` uniquely identifies a network connection.
6. **Bandwidth = max capacity. Throughput = actual rate. Latency = delay.**
7. **The internet uses packet switching** — more efficient than circuit switching. Tradeoff: need TCP for reliability.
8. **At every router hop:** IP stays the same, MAC gets rewritten to the next hop's MAC.
9. **Routers operate at L3 (IP). Switches operate at L2 (MAC).**
10. **Broadcast stays within the local network** — routers do not forward broadcast traffic.
