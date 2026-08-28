# Computer Networks — Complete Interview Cheat Sheet

> **Your single document for CN interview preparation.**
> Covers fundamentals through advanced topics, organized for both deep study and rapid revision.

---
---

# PART 1 — NETWORKING FUNDAMENTALS

---

## What Is a Computer Network?

A computer network is a collection of interconnected devices (computers, servers, phones, routers) that can exchange data.

**Why networks exist:**
- Share resources (files, printers, databases).
- Enable communication (email, chat, video).
- Provide centralized services (web servers, APIs, databases).

**Client-Server Model:**
- **Client** initiates a request (your browser).
- **Server** listens, processes, and responds (Google's web server).
- This is the dominant model of the internet — every HTTP request is a client asking a server for something.

**How devices communicate (high-level):**
1. Application generates data (e.g., an HTTP request).
2. Data is broken into smaller pieces (segments/packets).
3. Each piece is addressed (IP for destination, MAC for next hop).
4. Pieces travel through routers and switches across networks.
5. Destination reassembles pieces and delivers to the application.

---

## OSI Model `MUST KNOW`

The **Open Systems Interconnection** model is a 7-layer conceptual framework for how network communication works. Each layer has a specific responsibility and communicates with the layers directly above and below it.

| Layer | Name | Responsibility | Data Unit | Key Protocols/Tech | Key Devices |
|-------|------|---------------|-----------|-------------------|-------------|
| **7** | Application | User-facing services (HTTP, DNS, email) | Data/Message | HTTP, HTTPS, DNS, FTP, SMTP, SSH | — |
| **6** | Presentation | Data format, encryption, compression | Data | SSL/TLS, JPEG, ASCII, encryption | — |
| **5** | Session | Manage sessions/connections | Data | Session management, RPC | — |
| **4** | Transport | End-to-end reliable/unreliable delivery | Segment (TCP) / Datagram (UDP) | TCP, UDP, QUIC | — |
| **3** | Network | Logical addressing, routing across networks | Packet | IP, ICMP, ARP | Router |
| **2** | Data Link | Local delivery using MAC addresses, framing | Frame | Ethernet, Wi-Fi (802.11) | Switch, Bridge |
| **1** | Physical | Raw bit transmission over physical media | Bits | Cables, fiber, radio waves | Hub, Repeater |

### What happens at each layer:

**Layer 7 — Application:** Where your code lives. HTTP requests, DNS queries, SMTP emails. The application generates or consumes data.

**Layer 6 — Presentation:** Handles data formatting — encryption (TLS), compression (gzip), character encoding (UTF-8). In practice, often merged with the Application layer.

**Layer 5 — Session:** Manages sessions — establishing, maintaining, terminating connections. In practice, TCP handles most session-like functionality. Rarely discussed independently in interviews.

**Layer 4 — Transport:** End-to-end communication. TCP provides reliable, ordered delivery. UDP provides fast, unreliable delivery. This layer segments data and uses port numbers to identify processes.

**Layer 3 — Network:** Logical addressing (IP addresses) and routing. Determines the path a packet takes from source to destination across multiple networks. Routers operate here.

**Layer 2 — Data Link:** Local delivery within a single network segment. Uses MAC addresses. Frames data for transmission over the physical link. Switches operate here.

**Layer 1 — Physical:** The actual electrical/optical/wireless signals. Cables, fiber optics, Wi-Fi radio waves. Hubs and repeaters operate here.

> **Interview tip:** Layers 5, 6, 7 are often merged in practice. The TCP/IP model (4 layers) is what the internet actually uses. But know the OSI model for interviews — it's the standard reference.

---

## TCP/IP Model `MUST KNOW`

The practical model the internet actually uses. Four layers:

| TCP/IP Layer | OSI Equivalent | Protocols |
|-------------|---------------|-----------|
| **Application** | Application + Presentation + Session (L7+L6+L5) | HTTP, DNS, SMTP, FTP, SSH, TLS |
| **Transport** | Transport (L4) | TCP, UDP |
| **Internet** | Network (L3) | IP, ICMP, ARP |
| **Network Access** | Data Link + Physical (L2+L1) | Ethernet, Wi-Fi |

### OSI vs TCP/IP

| Feature | OSI | TCP/IP |
|---------|-----|--------|
| Layers | 7 | 4 |
| Purpose | Conceptual reference model | Practical implementation |
| Developed by | ISO | DARPA/DoD |
| Used in | Interviews, academia | The actual internet |
| Session/Presentation | Separate layers | Merged into Application |

**Why TCP/IP is more practical:** The internet was built on TCP/IP. OSI was designed after the fact as a conceptual model. In practice, layers 5 and 6 don't exist as distinct layers — TLS (encryption) is part of the application stack, and session management is handled by TCP or the application.

---

## Encapsulation and Decapsulation `MUST KNOW`

**Encapsulation** is the process of wrapping data with protocol headers as it moves down the stack. Each layer adds its own header.

```
Application Layer:  [HTTP Data]
                         ↓ add TCP header
Transport Layer:    [TCP Header | HTTP Data]               = Segment
                         ↓ add IP header
Network Layer:      [IP Header | TCP Header | HTTP Data]   = Packet
                         ↓ add Ethernet header + trailer
Data Link Layer:    [Eth Header | IP Header | TCP Header | HTTP Data | FCS]  = Frame
                         ↓ convert to signals
Physical Layer:     10110100101110010101...                 = Bits
```

**Decapsulation** is the reverse — the receiving machine strips headers layer by layer, moving up the stack until the application gets the original data.

### What each header contains:

| Layer | Header Added | Key Fields |
|-------|-------------|------------|
| Transport | TCP/UDP header | Source port, destination port, sequence number, flags, window size |
| Network | IP header | Source IP, destination IP, TTL, protocol |
| Data Link | Ethernet header + FCS trailer | Source MAC, destination MAC, EtherType, Frame Check Sequence |

**Key insight for interviews:** When a packet travels across routers, **IP addresses stay the same** (end-to-end) but **MAC addresses change at every hop** (the frame is rewritten at each router with the next hop's MAC). This is a critical distinction.

---

## Important Terminology

| Term | Layer | Description |
|------|-------|-------------|
| **Data / Message** | Application (L7) | Raw application data |
| **Segment** | Transport (L4) | TCP data unit — includes TCP header + data |
| **Datagram** | Transport (L4) | UDP data unit — includes UDP header + data |
| **Packet** | Network (L3) | IP data unit — includes IP header + segment/datagram |
| **Frame** | Data Link (L2) | Ethernet data unit — includes Ethernet header + packet + FCS |
| **Bits** | Physical (L1) | Raw 0s and 1s on the wire/air |

---

## Addressing `MUST KNOW`

Three types of addresses are used, each at a different layer, each solving a different problem.

### MAC Address (Layer 2)

- **48-bit** hardware address, written as `AA:BB:CC:DD:EE:FF`.
- Burned into the NIC (Network Interface Card) by the manufacturer.
- **Scope: Local only** — used for delivery within a single network segment (LAN).
- **Changes at every hop** — when a packet crosses a router, the frame gets a new source/destination MAC.
- Broadcast MAC: `FF:FF:FF:FF:FF:FF` — reaches every device on the LAN.

### IP Address (Layer 3)

- **32-bit** (IPv4) or **128-bit** (IPv6) logical address, e.g., `192.168.1.10`.
- Assigned by software (DHCP or manual configuration).
- **Scope: End-to-end** — stays the same from source to destination (ignoring NAT).
- Used by routers to forward packets across networks.

### Port Number (Layer 4)

- **16-bit** number (0–65535) identifying a specific process or service on a machine.
- Tells the OS which application should receive the data.
- Well-known ports: HTTP=80, HTTPS=443, DNS=53, SSH=22.
- Ephemeral ports (49152–65535): OS assigns these to client connections.

### Why all three are needed

| Address | Answers | Scope |
|---------|---------|-------|
| **MAC** | "Which device on **this local link** gets this frame?" | Hop-by-hop (local) |
| **IP** | "Which machine **on the internet** is the final destination?" | End-to-end (global) |
| **Port** | "Which **application/process** on that machine?" | Process-level |

**Real example:**

Your browser (port 52431) sends an HTTP request to `google.com` (IP `142.250.190.46`, port 443):

```
Your Machine:     IP=192.168.1.10, MAC=AA:AA:AA:AA:AA:AA, Port=52431
Router (gateway): IP=192.168.1.1,  MAC=BB:BB:BB:BB:BB:BB
Google Server:    IP=142.250.190.46, MAC=?? (you never know it), Port=443

Frame leaving your machine:
  Src MAC: AA:AA (your MAC)
  Dst MAC: BB:BB (router's MAC ← NOT Google's!)
  Src IP:  192.168.1.10
  Dst IP:  142.250.190.46
  Src Port: 52431
  Dst Port: 443
```

The MAC is the router's because the frame only needs to reach the next hop. The IP is Google's because the packet needs to reach the final destination.

---

## Network Performance

| Metric | What It Measures | Example |
|--------|-----------------|---------|
| **Bandwidth** | Maximum data rate a link can carry | "100 Mbps Ethernet" |
| **Throughput** | Actual data rate achieved (≤ bandwidth) | "Getting 45 Mbps on a 100 Mbps link" |
| **Latency** | Time delay for data to travel from A to B | "20ms round-trip to the server" |
| **Jitter** | Variation in latency | "Latency fluctuates 10–50ms" (bad for video calls) |
| **Packet Loss** | Percentage of packets that don't arrive | "0.5% packet loss" (causes retransmission in TCP) |

**Bandwidth ≠ throughput.** Bandwidth is the pipe size; throughput is how much water actually flows through it (affected by congestion, protocol overhead, distance).

---

## Communication Types

| Type | Destination | Example |
|------|------------|---------|
| **Unicast** | One specific device | HTTP request to a web server |
| **Broadcast** | All devices on the network | ARP request ("Who has 192.168.1.1?") |
| **Multicast** | A group of interested devices | Live video streaming to subscribers |

---

## Circuit Switching vs Packet Switching

| Feature | Circuit Switching | Packet Switching |
|---------|------------------|-----------------|
| How it works | Dedicated path reserved for entire communication | Data split into packets, each routed independently |
| Bandwidth | Reserved (wasted if idle) | Shared (efficient) |
| Delay | Constant | Variable (depends on congestion) |
| Reliability | Guaranteed path | Packets can be lost, reordered |
| Example | Traditional telephone (PSTN) | The Internet |

**Why the Internet uses packet switching:** Efficiency. A dedicated path (circuit switching) wastes bandwidth when no data is flowing. Packet switching shares links among all users, maximizing utilization. TCP provides reliability on top of this unreliable packet-switched network.

---
---

# PART 2 — PHYSICAL + DATA LINK LAYER

---

## Physical Layer (Brief)

The physical layer transmits raw **bits** (0s and 1s) over physical media:
- **Copper cables** — electrical signals (Ethernet, Cat5/Cat6).
- **Fiber optic** — light pulses (long-distance, high-speed).
- **Wireless** — radio waves (Wi-Fi, cellular).

Key concept: **Bit rate** — how many bits per second the medium can carry.

Not heavily asked in SDE interviews. Know it exists at L1.

---

## Data Link Layer `MUST KNOW`

**Purpose:** Reliable delivery of frames between two **directly connected** devices on the same network segment.

**Key responsibilities:**
- **Framing** — wraps packets into frames with headers/trailers.
- **MAC addressing** — identifies devices on the local link.
- **Error detection** — CRC in the frame trailer detects corrupted frames.
- **Media access** — coordinates access to the shared medium (CSMA/CD for Ethernet, CSMA/CA for Wi-Fi).

---

## Ethernet `MUST KNOW`

The dominant Layer 2 technology for wired LANs.

### Ethernet Frame Structure

```
┌──────────────┬──────────────┬───────────┬─────────────────────────┬─────┐
│ Dst MAC (6B) │ Src MAC (6B) │ Type (2B) │      Payload (46-1500B) │ FCS │
└──────────────┴──────────────┴───────────┴─────────────────────────┴─────┘
```

| Field | Size | Purpose |
|-------|------|---------|
| Destination MAC | 6 bytes | Who this frame is for |
| Source MAC | 6 bytes | Who sent this frame |
| EtherType | 2 bytes | What protocol is in the payload (0x0800 = IPv4, 0x0806 = ARP) |
| Payload | 46–1500 bytes | The IP packet |
| FCS | 4 bytes | Frame Check Sequence — CRC for error detection |

**MTU (Maximum Transmission Unit):** 1500 bytes — the maximum payload an Ethernet frame can carry. Packets larger than this must be fragmented.

---

## Hubs and Switches `MUST KNOW`

### Hub (Layer 1)

- A **dumb repeater** with multiple ports.
- Receives a signal on one port → repeats it out **all other ports**.
- All devices share one **collision domain** — if two devices transmit simultaneously, signals collide.
- **Obsolete.** Replaced by switches.

### Switch (Layer 2)

- An **intelligent** device that forwards frames based on **MAC addresses**.
- Maintains a **MAC address table** (CAM table) mapping MAC addresses to ports.
- Each port is its own **collision domain** — no collisions in modern switched networks.

### How a Switch Learns MAC Addresses

1. Frame arrives on port 1 with source MAC `AA:AA:AA`.
2. Switch records: `AA:AA:AA → Port 1` in its MAC table.
3. Switch checks destination MAC in the table.
4. **If found** → forward frame only to that specific port (unicast forwarding).
5. **If NOT found** (unknown unicast) → **flood** the frame out all ports except the source port.
6. When the destination replies, the switch learns its MAC too.

Over time, the switch builds a complete table and forwards precisely.

### Hub vs Switch

| Feature | Hub | Switch |
|---------|-----|--------|
| Layer | L1 (Physical) | L2 (Data Link) |
| Intelligence | None — repeats to all | Learns MACs, forwards selectively |
| Collision domain | 1 shared (all ports) | 1 per port (no collisions) |
| Bandwidth | Shared among all | Dedicated per port |
| Modern use | Obsolete | Standard |

---

## Collision Domain vs Broadcast Domain

| | Collision Domain | Broadcast Domain |
|-|-----------------|-----------------|
| Definition | Area where simultaneous transmissions cause collisions | Area where broadcast frames are received |
| Separated by | **Switch** (each port = separate collision domain) | **Router** (blocks broadcasts) |
| Hub effect | 1 collision domain for entire hub | 1 broadcast domain |
| Switch effect | 1 collision domain per port | 1 broadcast domain for entire switch |
| Router effect | N/A | Each interface = separate broadcast domain |

**Key insight:** A switch eliminates collisions but does NOT block broadcasts. A router blocks broadcasts — it separates broadcast domains.

---

## ARP — Address Resolution Protocol `MUST KNOW`

### Why ARP Exists

When your computer wants to send data to another device on the **same LAN**, it knows the destination's **IP address** but needs the **MAC address** to create an Ethernet frame. ARP resolves IP → MAC.

### How ARP Works

**Step-by-step: Computer A (192.168.1.10) wants to send data to Computer B (192.168.1.20) on the same LAN.**

1. **A checks its ARP cache** — does it already know B's MAC? If yes → use it.
2. **ARP Request (broadcast):** A sends a broadcast frame to `FF:FF:FF:FF:FF:FF`:
   - "Who has IP `192.168.1.20`? Tell `192.168.1.10`"
3. **All devices receive it.** Only B recognizes its own IP.
4. **ARP Reply (unicast):** B responds directly to A:
   - "I am `192.168.1.20`, my MAC is `CC:CC:CC:CC:CC:CC`"
5. **A caches** the mapping: `192.168.1.20 → CC:CC:CC:CC:CC:CC`.
6. **A sends the data** in an Ethernet frame with B's MAC as destination.

```
A (192.168.1.10)                                B (192.168.1.20)
  |                                               |
  |── ARP Request (broadcast) ──────────────────>| (all devices)
  |   "Who has 192.168.1.20?"                     |
  |                                               |
  |<────────────────── ARP Reply (unicast) ───────|
  |   "192.168.1.20 is at CC:CC:CC:CC:CC:CC"     |
  |                                               |
  |── Data Frame (to CC:CC:CC) ────────────────>  |
```

### What if Computer B is on Another Network?

If the destination IP is **NOT on the local subnet** (different network), the host **does NOT ARP for the remote server**. Instead:

1. Host compares destination IP with its own subnet → different network.
2. Host needs to send the packet to its **default gateway** (router).
3. Host ARPs for the **router's MAC address** (not the remote server's).
4. Frame is sent with: **destination MAC = router's MAC, destination IP = remote server's IP**.
5. Router receives the frame, strips the L2 header, reads the destination IP, consults its routing table, and forwards the packet to the next hop — rewriting the MAC addresses again.

**This is a critical interview point:** When sending to a remote host, ARP resolves the **gateway's** MAC, not the final destination's MAC. MAC addresses are local (hop-by-hop); IP addresses are global (end-to-end).

---

## VLAN `GOOD TO KNOW`

**Virtual LAN** — logically segments a physical switch into multiple separate broadcast domains.

**Why VLANs exist:**
- **Security:** Isolate traffic between departments (HR can't see engineering traffic).
- **Broadcast reduction:** Broadcasts only reach devices in the same VLAN, not the entire switch.
- **Flexibility:** Group devices logically regardless of physical location.

**How it works:** The switch tags frames with a VLAN ID (802.1Q tag). Frames stay within their VLAN unless a router routes between them (inter-VLAN routing).

**Interview point:** "A VLAN creates a logical broadcast domain on a switch. Devices in different VLANs can't communicate at Layer 2 — they need a router, just like separate physical networks."

---
---

# PART 3 — NETWORK LAYER

---

## IP Addressing `MUST KNOW`

### IPv4

- **32-bit** address, written as four octets: `192.168.1.10`.
- Split into **network portion** (identifies the network) and **host portion** (identifies the device on that network).
- The **subnet mask** defines where the split happens.

```
IP:          192.168.1.10
Subnet Mask: 255.255.255.0  (/24)

Network:     192.168.1.0    (first 24 bits)
Host:                 .10   (last 8 bits)
```

---

## Public vs Private IP `MUST KNOW`

| Type | Routable on Internet? | Assigned By | Purpose |
|------|-----------------------|-------------|---------|
| **Public** | Yes — globally unique | ISP / Registry (IANA) | Internet-facing communication |
| **Private** | No — reused in different networks | Anyone (internal use) | Internal network communication |

### Private IP Ranges

| Range | CIDR | Class |
|-------|------|-------|
| `10.0.0.0` – `10.255.255.255` | 10.0.0.0/8 | A |
| `172.16.0.0` – `172.31.255.255` | 172.16.0.0/12 | B |
| `192.168.0.0` – `192.168.255.255` | 192.168.0.0/16 | C |

**Why private IPs exist:** IPv4 has only ~4.3 billion addresses — not enough for every device. Private IPs let millions of devices reuse the same addresses internally, with NAT providing internet access through shared public IPs.

---

## Special Addresses

| Address | Purpose |
|---------|---------|
| `127.0.0.1` | **Loopback** — refers to the local machine itself. `ping 127.0.0.1` tests your own TCP/IP stack. |
| `x.x.x.0` (network address) | Identifies the **network itself** (not assignable to a host). |
| `x.x.x.255` (broadcast address) | **Broadcast** — reaches all hosts on the subnet. |
| `0.0.0.0` | "Any address" or "all interfaces" — used in routing as the default route. |
| `169.254.x.x` | **APIPA** — self-assigned when DHCP fails. |

---

## Subnetting `MUST KNOW`

### Why Subnetting Exists

- **Efficient IP allocation:** A /24 gives 254 hosts. If you only need 30, use a /27 (30 hosts) instead.
- **Network organization:** Divide a large network into smaller, manageable subnets (one per department, floor, etc.).
- **Broadcast control:** Smaller subnets = smaller broadcast domains = less noise.

### CIDR (Classless Inter-Domain Routing) Notation

`192.168.1.0/24` — the `/24` means the first 24 bits are the network portion.

### Subnetting Reference Table

| CIDR | Subnet Mask | Total Addresses | Usable Hosts |
|------|------------|-----------------|--------------|
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 2 |

**Formulas:**
- Total addresses = 2^(32 − prefix)
- Usable hosts = 2^(32 − prefix) − 2 (subtract network address and broadcast address)

### Worked Example

**Given:** `192.168.10.50/26`

1. **Block size:** 2^(32-26) = 2^6 = 64 addresses per subnet.
2. **Subnets start at:** .0, .64, .128, .192.
3. **50 falls in the .0 block** (0–63).
4. **Network address:** `192.168.10.0`
5. **Broadcast address:** `192.168.10.63`
6. **Usable range:** `192.168.10.1` – `192.168.10.62` (62 hosts)

---

## Routers `MUST KNOW`

### What a Router Does

- Connects **different networks** together.
- Operates at **Layer 3** — forwards packets based on **IP addresses**.
- Maintains a **routing table** — maps destination networks to next hops and outgoing interfaces.
- **Separates broadcast domains** — broadcasts don't cross routers.

### Routing Table

| Destination Network | Next Hop | Interface |
|--------------------|----------|-----------|
| 192.168.1.0/24 | Directly connected | eth0 |
| 10.0.0.0/8 | 172.16.0.1 | eth1 |
| 0.0.0.0/0 (default) | 203.0.113.1 | eth2 |

### How a Router Forwards a Packet

1. Receive frame, strip L2 header.
2. Read destination IP from the IP header.
3. Search routing table using **longest prefix match** — the most specific matching route wins.
4. Decrement **TTL** by 1 (drop packet if TTL reaches 0).
5. Create a new L2 frame with the next hop's MAC address (via ARP).
6. Send the frame out the appropriate interface.

### Longest Prefix Match

If a packet is destined for `10.1.2.3` and the routing table has:
- `10.0.0.0/8` → via Router A
- `10.1.0.0/16` → via Router B
- `10.1.2.0/24` → via Router C

**Router C wins** — `/24` is the longest (most specific) match.

### Default Route

`0.0.0.0/0` — matches everything. Used when no more specific route exists. Acts as the "if nothing else matches, send it here" rule.

---

## Default Gateway `MUST KNOW`

The **router** your device sends packets to when the destination is **not on the local network**. Usually `192.168.1.1` or similar.

**How it works:**
1. Your machine wants to reach `142.250.190.46` (Google).
2. It compares with its own subnet (e.g., `192.168.1.0/24`) — Google is NOT local.
3. Sends the packet to the default gateway's IP.
4. But first, ARPs for the gateway's **MAC address** to build the Ethernet frame.
5. Gateway router receives the frame and forwards the packet toward Google.

---

## NAT (Network Address Translation) `MUST KNOW`

### Why NAT Exists

- IPv4 doesn't have enough addresses for every device.
- NAT lets **multiple devices** with private IPs share a **single public IP** to access the internet.
- Your home router does NAT — all your devices (phone, laptop, TV) share your ISP-assigned public IP.

### How NAT Works

1. Device `192.168.1.10:52431` sends a packet to `142.250.190.46:443`.
2. Router receives it, **rewrites** source IP to the router's public IP (`203.0.113.5`) and assigns a unique source port (`40001`).
3. Router records the mapping in the **NAT table**: `192.168.1.10:52431 ↔ 203.0.113.5:40001`.
4. Packet goes to Google with source `203.0.113.5:40001`.
5. Google responds to `203.0.113.5:40001`.
6. Router receives the response, looks up `40001` in the NAT table → `192.168.1.10:52431`.
7. Rewrites destination back to private IP and delivers to the device.

### PAT (Port Address Translation)

The most common form of NAT. Uses **port numbers** to multiplex many private IPs through one public IP. Each internal connection gets a unique port on the public side.

```
Device A (192.168.1.10:52431)  →  Public IP 203.0.113.5:40001  →  google.com:443
Device B (192.168.1.11:48000)  →  Public IP 203.0.113.5:40002  →  google.com:443
Device C (192.168.1.12:55123)  →  Public IP 203.0.113.5:40003  →  github.com:443
```

All three devices share one public IP. The router tracks each connection by port.

---

## TTL (Time to Live) `MUST KNOW`

- A field in the **IP header** set by the sender (commonly 64 or 128).
- **Decremented by 1** at every router hop.
- When TTL reaches **0**, the router **drops the packet** and sends an **ICMP Time Exceeded** message back to the sender.

**Why it exists:** Prevents packets from looping forever in the network due to routing errors. Without TTL, a routing loop would circulate packets indefinitely.

---

## ICMP `GOOD TO KNOW`

**Internet Control Message Protocol** — used for network diagnostics and error reporting. Not for data transfer.

**Key ICMP messages:**

| Type | Name | Used By |
|------|------|---------|
| Echo Request / Echo Reply | Ping | `ping` command |
| Destination Unreachable | Error | Router can't deliver a packet |
| Time Exceeded | TTL expired | `traceroute` command |

### Ping

Sends an ICMP Echo Request to a target → receives Echo Reply. Measures round-trip time and checks if the host is reachable.

### Traceroute

Exploits TTL to discover each router hop along the path:
1. Send packet with TTL=1 → first router drops it, sends ICMP Time Exceeded (reveals router 1's IP).
2. Send packet with TTL=2 → second router drops it (reveals router 2's IP).
3. Continue incrementing TTL until the destination is reached.

---

## IPv6 `GOOD TO KNOW`

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Address size | 32 bits | 128 bits |
| Address format | Dotted decimal: `192.168.1.1` | Hex: `2001:0db8:85a3::8a2e:0370:7334` |
| Address space | ~4.3 billion | ~3.4 × 10^38 |
| NAT needed? | Yes (address scarcity) | No (enough addresses for every device) |
| Header | Variable, complex | Fixed 40 bytes, simplified |

**Why IPv6 exists:** IPv4 address exhaustion. There are more internet-connected devices than IPv4 addresses. IPv6 provides virtually unlimited addresses.

**Interview point:** Know that IPv6 exists and why. Don't go deep into IPv6 internals for SDE interviews.

---
---

# PART 4 — TRANSPORT LAYER

---

## Ports `MUST KNOW`

A **16-bit number** (0–65535) identifying a specific process or service on a machine.

### Port Ranges

| Range | Name | Purpose | Examples |
|-------|------|---------|---------|
| 0–1023 | Well-known | Reserved for standard services | HTTP=80, HTTPS=443, DNS=53, SSH=22 |
| 1024–49151 | Registered | Application-specific | MySQL=3306, PostgreSQL=5432, Redis=6379 |
| 49152–65535 | Ephemeral | OS-assigned to client connections | Your browser uses a random port here |

### Important Port Numbers

| Port | Protocol | Service |
|------|----------|---------|
| 22 | TCP | SSH |
| 25 | TCP | SMTP (email sending) |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |
| 3306 | TCP | MySQL |
| 5432 | TCP | PostgreSQL |
| 6379 | TCP | Redis |

**Note:** TCP and UDP port spaces are **independent**. TCP port 53 and UDP port 53 are different.

---

## Sockets `MUST KNOW`

### What a Socket Is

A **socket** is a communication endpoint, defined by:

```
Socket = (IP Address, Port Number)
```

It's the interface between the application layer and the transport layer — how your code talks to the network.

### Connection Identity

A TCP connection is uniquely identified by a **5-tuple:**

```
(Source IP, Source Port, Destination IP, Destination Port, Protocol)
```

This is why a server on port 443 can handle **thousands** of simultaneous connections — each connection has a unique combination of source IP + source port.

### Socket Lifecycle

**Server side:**
```
socket()   → Create a socket
bind()     → Attach to IP + port (e.g., 0.0.0.0:8080)
listen()   → Mark socket as ready for incoming connections
accept()   → Block until a client connects → return a NEW socket for that client
recv/send  → Exchange data
close()    → Close connection
```

**Client side:**
```
socket()   → Create a socket
connect()  → Initiate TCP handshake to server (OS assigns ephemeral port)
send/recv  → Exchange data
close()    → Close connection (triggers TCP FIN)
```

**Critical point:** `accept()` returns a **NEW socket** for each client. The original listening socket continues accepting new connections. This is how one server port serves many clients.

---
---

# PART 5 — TCP

---

## What is TCP? `MUST KNOW`

**Transmission Control Protocol** — a connection-oriented, reliable transport protocol.

| Property | What It Means |
|----------|--------------|
| **Connection-oriented** | Must establish a connection (3-way handshake) before data transfer |
| **Reliable** | Guarantees delivery — retransmits lost data |
| **Ordered** | Data arrives in the order it was sent (sequence numbers) |
| **Byte stream** | Application sees a continuous stream of bytes, not individual messages |
| **Full-duplex** | Both sides can send and receive simultaneously |
| **Flow-controlled** | Receiver tells sender how fast to send (prevents overwhelming) |
| **Congestion-controlled** | Sender adapts rate to network conditions (prevents congestion) |

---

## TCP Segment Structure `MUST KNOW`

```
┌────────────────────────────────────────────────────────────┐
│                    TCP SEGMENT HEADER                       │
├──────────────────────┬─────────────────────────────────────┤
│  Source Port (16)    │  Destination Port (16)              │
├──────────────────────┴─────────────────────────────────────┤
│                  Sequence Number (32)                       │
├────────────────────────────────────────────────────────────┤
│               Acknowledgment Number (32)                   │
├──────┬───────┬────────────┬────────────────────────────────┤
│Offset│Reservd│  Flags     │      Window Size (16)          │
│ (4)  │ (3)   │SYN ACK FIN │                                │
├──────┴───────┴────────────┴────────────────────────────────┤
│    Checksum (16)     │    Urgent Pointer (16)              │
├──────────────────────┴─────────────────────────────────────┤
│                     Options (variable)                      │
├────────────────────────────────────────────────────────────┤
│                        DATA                                │
└────────────────────────────────────────────────────────────┘
```

### Key Fields

| Field | Purpose |
|-------|---------|
| **Source/Dest Port** | Identifies sending and receiving applications |
| **Sequence Number** | Position of the first byte in this segment within the byte stream |
| **ACK Number** | The next byte the sender expects to receive (acknowledges all bytes before this) |
| **Flags** | SYN (initiate), ACK (acknowledge), FIN (terminate), RST (reset), PSH (push) |
| **Window Size** | Receiver's available buffer space (flow control) |

---

## TCP 3-Way Handshake `MUST KNOW`

Establishes a TCP connection and synchronizes sequence numbers.

```
Client                                    Server
  |                                         |
  |── SYN (seq=x) ───────────────────────>|    Client: "I want to connect. My ISN is x."
  |                                         |
  |<──── SYN-ACK (seq=y, ack=x+1) ────────|    Server: "OK. My ISN is y. I got your x."
  |                                         |
  |── ACK (seq=x+1, ack=y+1) ───────────>|    Client: "I got your y. Let's go."
  |                                         |
  |═══════ Connection ESTABLISHED ═════════|
```

**Step 1 — SYN:** Client sends SYN flag with its Initial Sequence Number (ISN = x). Client enters `SYN_SENT` state.

**Step 2 — SYN-ACK:** Server responds with SYN + ACK. Its own ISN = y, and it acknowledges x+1 (meaning "I received up to x, send me x+1 next"). Server enters `SYN_RECEIVED` state.

**Step 3 — ACK:** Client acknowledges y+1. Both sides enter `ESTABLISHED` state.

**Cost:** 1 RTT (round-trip time) before data can flow.

### Why 3 Steps, Not 2?

With only 2 steps (SYN → SYN-ACK), the server confirms the client's ISN but the client never confirms the server's ISN. The server starts sending data without knowing if the client received its ISN. This can lead to:
- Old/duplicate SYN packets opening ghost connections.
- Client and server disagreeing on sequence numbers.

The third ACK confirms both sides agree on each other's ISNs.

### Edge Cases

**SYN lost:** Client never gets SYN-ACK → retransmits SYN with exponential backoff (1s, 2s, 4s...). After max retries → connection fails.

**SYN-ACK lost:** Client retransmits SYN (looks like lost SYN). Server may retransmit SYN-ACK. Eventually they sync or both timeout.

**Final ACK lost:** Server stays in SYN_RECEIVED, retransmits SYN-ACK. Client is already ESTABLISHED — if it sends data, those segments carry ACK flags that complete the handshake for the server.

---

## TCP Reliability `MUST KNOW`

TCP provides reliable delivery over an unreliable IP network.

### Sequence Numbers and ACKs

- Each byte in the stream has a **sequence number**.
- The receiver sends **ACKs** indicating the next byte it expects.
- ACK number = "I've received everything up to this byte, send me the next one."
- **Cumulative ACKs:** ACK 5000 means "I have bytes 0–4999, send 5000 next."

### Retransmission (How TCP Handles Loss)

**Timeout-based:** If no ACK arrives within the **Retransmission Timeout (RTO)**, the sender retransmits the segment. RTO is calculated from measured RTT.

**Fast Retransmit:** If the sender receives **3 duplicate ACKs** (the receiver keeps ACKing the same byte because it's missing a segment), the sender retransmits immediately without waiting for timeout. This is faster because the network is still partially working.

### Out-of-Order Segments

The receiver **buffers** out-of-order segments and sends duplicate ACKs for the missing byte. When the missing segment arrives, TCP reorders everything and delivers to the application in order. The application never sees out-of-order data.

---

## Sliding Window `MUST KNOW`

### Why It Exists

Without a sliding window, TCP would send one segment, wait for ACK, send the next — terribly slow (like Stop-and-Wait). The sliding window lets the sender have **multiple segments in flight** simultaneously.

### How It Works

The window defines how many bytes the sender can transmit before needing an ACK. As ACKs arrive, the window "slides" forward, allowing more data to be sent.

**Effective window = min(rwnd, cwnd)** — the smaller of the receiver's advertised window and the sender's congestion window.

---

## Flow Control `MUST KNOW`

**Problem:** The sender might transmit data faster than the receiver can process it → receiver buffer overflows → data loss.

**Solution:** The receiver advertises its available buffer space via the **receive window (rwnd)** field in every ACK.

**How it works:**
1. Receiver has buffer space = 64KB → advertises `rwnd = 64KB`.
2. Sender limits in-flight data to 64KB.
3. As receiver processes data, buffer space frees up → rwnd increases.
4. If receiver is overwhelmed → rwnd decreases (can reach 0 = **zero window**, sender stops).

**Key insight:** Flow control protects the **receiver** from being overwhelmed by the sender.

---

## Congestion Control `MUST KNOW`

**Problem:** If the sender transmits too fast, the **network** (routers, links) becomes congested → packets dropped → retransmissions → more congestion (congestion collapse).

**Solution:** The sender maintains a **congestion window (cwnd)** and adjusts its sending rate based on network feedback.

### Congestion Control Phases

**1. Slow Start** (exponential growth)
- cwnd starts at 1 MSS (Maximum Segment Size).
- cwnd **doubles** every RTT (1 → 2 → 4 → 8 → 16...).
- "Slow" only means it starts slow — growth is actually exponential.
- Continues until cwnd reaches **ssthresh** (slow start threshold).

**2. Congestion Avoidance** (linear growth)
- After ssthresh, cwnd grows by **1 MSS per RTT** (additive increase).
- This is the AIMD (Additive Increase, Multiplicative Decrease) phase.

**3. Loss Detection → Response**

| Loss Signal | Severity | Response |
|-------------|----------|----------|
| **3 Duplicate ACKs** | Mild (some packets got through) | ssthresh = cwnd/2, cwnd = ssthresh. **Fast Recovery.** |
| **Timeout** | Severe (nothing is getting through) | ssthresh = cwnd/2, cwnd = **1 MSS**. Back to **Slow Start.** |

```
cwnd
 ^
 │        Slow Start          Congestion Avoidance
 │       (exponential)              (linear)
 │            /                    /
 │           /                   /
 │          /                  /     ← 3 dup ACKs: cwnd halved
 │         /                 / |
 │        /                /   |
 │       /               /    ↓ Fast Recovery
 │      /              /     /
 │     /             /      /
 │    /            /       /
 │   /           /        /     ← Timeout: cwnd = 1
 │  /          /         / |
 │ /         /          /  ↓ Slow Start again
 │/________/___________/____________________> Time
        ssthresh
```

---

## Flow Control vs Congestion Control — The Critical Distinction

| | Flow Control | Congestion Control |
|-|---|---|
| **Prevents overwhelming** | The **receiver** | The **network** |
| **Controlled by** | Receiver (advertises rwnd) | Sender (adjusts cwnd) |
| **Variable** | rwnd (receive window) | cwnd (congestion window) |
| **Mechanism** | Window size in TCP header | Slow start, AIMD, fast recovery |
| **Feedback source** | Receiver → sender | Packet loss signals (timeout, dup ACKs) |

**Effective sending window = min(rwnd, cwnd)**

The sender can never send more than the smaller of what the receiver can handle (rwnd) and what the network can handle (cwnd).

---

## TCP Connection Termination `MUST KNOW`

TCP uses a **4-way handshake** to close a connection because TCP is **full-duplex** — each direction must be closed independently.

```
Client                                    Server
  |                                         |
  |── FIN (seq=u) ─────────────────────── >|    "I'm done sending."
  |                                         |
  |<──── ACK (ack=u+1) ────────────────────|    "Got it. I may still send."
  |                                         |
  |     (Server may send remaining data)    |
  |                                         |
  |<──── FIN (seq=v) ──────────────────────|    "I'm done sending too."
  |                                         |
  |── ACK (ack=v+1) ─────────────────────>|    "Got it. Connection closed."
  |                                         |
  [Client enters TIME_WAIT]
```

### Why 4 Messages?

TCP is full-duplex — data flows independently in both directions. The first FIN+ACK closes one direction. The second FIN+ACK closes the other. Each side must independently signal it's done sending.

### Half-Close

After the first FIN+ACK exchange, one direction is closed but the other is still open. The server can continue sending data to the client. This is called a **half-close**.

### TIME_WAIT `MUST KNOW`

After sending the final ACK, the initiator enters **TIME_WAIT** for **2×MSL** (Maximum Segment Lifetime, typically 120 seconds).

**Why TIME_WAIT exists:**

1. **Ensure the final ACK arrives.** If it's lost, the peer retransmits its FIN, and the host in TIME_WAIT can resend the ACK.
2. **Allow old duplicate packets to expire.** Prevents stale packets from a closed connection being confused with a new connection on the same port pair.

---
---

# PART 6 — UDP

---

## What is UDP? `MUST KNOW`

**User Datagram Protocol** — a connectionless, unreliable transport protocol.

| Property | What It Means |
|----------|--------------|
| **Connectionless** | No handshake — just send |
| **Unreliable** | No guaranteed delivery, no retransmission |
| **Unordered** | Packets may arrive out of order |
| **Datagram-oriented** | Each `send()` = one complete message (not a byte stream) |
| **No flow control** | Sender doesn't adapt to receiver's capacity |
| **No congestion control** | Sender doesn't adapt to network conditions |
| **Low overhead** | 8-byte header (vs TCP's 20+ bytes) |

### UDP Header

```
┌──────────────────────┬─────────────────────────┐
│  Source Port (16)    │  Destination Port (16)  │
├──────────────────────┼─────────────────────────┤
│    Length (16)        │    Checksum (16)        │
└──────────────────────┴─────────────────────────┘
                    8 bytes total
```

### Why UDP Can Be Faster/Lower Latency

- No handshake delay (no 3-way handshake before sending data).
- No retransmission delay (no waiting for lost packets — just move on).
- No congestion/flow control overhead.
- Smaller header (8 bytes vs 20+ bytes).

**Important nuance:** UDP is not inherently "faster" in every scenario. It has less **protocol overhead** and avoids TCP's connection/reliability mechanisms. For bulk data transfer, TCP's flow control and congestion control actually achieve higher throughput. UDP's advantage is **lower latency** — critical for real-time applications where late data is worse than lost data.

### UDP Use Cases

- **DNS** — small queries, speed matters, can retry at application level.
- **Video/audio streaming** — late retransmitted frames are useless.
- **Online gaming** — latency matters more than perfect delivery.
- **VoIP** — real-time voice can't wait for retransmissions.
- **QUIC/HTTP/3** — builds reliability on top of UDP with per-stream control.

---

## TCP vs UDP `MUST KNOW`

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Connection-oriented (3-way handshake) | Connectionless |
| **Reliability** | Reliable (ACK, retransmission) | Unreliable (best-effort) |
| **Ordering** | Ordered (sequence numbers) | No ordering guarantee |
| **Flow Control** | Yes (rwnd) | No |
| **Congestion Control** | Yes (cwnd, slow start, AIMD) | No |
| **Data Model** | Byte stream | Datagram/message |
| **Header Size** | 20–60 bytes | 8 bytes |
| **Speed/Latency** | Higher latency (handshake, retransmission) | Lower latency |
| **Use Cases** | HTTP, SSH, email, file transfer | DNS, streaming, gaming, VoIP, QUIC |

**When to use TCP:** When you need guaranteed, ordered delivery and can tolerate slightly higher latency (web, APIs, file transfer, email).

**When to use UDP:** When low latency is critical and you can tolerate some loss (real-time streaming, gaming, VoIP), or when you want to build your own reliability on top (QUIC).

---
---

# PART 7 — APPLICATION LAYER: DNS

---

## DNS — Domain Name System `MUST KNOW`

### What It Is

DNS translates human-readable **domain names** (like `google.com`) into **IP addresses** (like `142.250.190.46`) that computers use for routing.

### Why It Exists

Humans remember names, computers use numbers. Without DNS, you'd memorize IP addresses for every website.

### Key Facts

- Application-layer protocol.
- Uses **UDP port 53** (for queries) and **TCP port 53** (for zone transfers and large responses).
- DNS is a **distributed, hierarchical** database — no single server has all the answers.

---

## DNS Hierarchy

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

| Component | Role |
|-----------|------|
| **Root DNS Servers** | 13 logical groups (anycast). Know where TLD servers are. |
| **TLD Servers** | Handle `.com`, `.org`, `.net`, etc. Know where authoritative servers are. |
| **Authoritative Servers** | Hold the **actual DNS records** for a domain. The final stop. |
| **Recursive Resolver** | Your first contact (ISP or `8.8.8.8`). Does all the work on your behalf. Caches results. |

---

## DNS Resolution — Complete Flow `MUST KNOW`

**How `google.com` becomes `142.250.190.46`:**

```
Browser                  OS              Recursive Resolver    Root    .com TLD   google.com Auth
  |                      |                     |                |         |            |
  |─ Check browser cache |                     |                |         |            |
  |  (miss)              |                     |                |         |            |
  |─ Query OS ──────────>|                     |                |         |            |
  |                      |─ Check OS cache     |                |         |            |
  |                      |  (miss)             |                |         |            |
  |                      |─ Query resolver ──>|                |         |            |
  |                      |                     |─ Check cache   |         |            |
  |                      |                     |  (miss)        |         |            |
  |                      |                     |─ Query root ─>|         |            |
  |                      |                     |<─ "Ask .com"  |         |            |
  |                      |                     |─ Query .com ────────── >|            |
  |                      |                     |<─ "Ask google auth" ───|            |
  |                      |                     |─ Query auth ──────────────────────>  |
  |                      |                     |<─ "142.250.190.46" ────────────────  |
  |                      |<─ 142.250.190.46 ──|                |         |            |
  |<─ 142.250.190.46 ───|                     |                |         |            |
  |                      |                     |                |         |            |
  [All levels cache the result with TTL]
```

---

## Recursive vs Iterative Queries `MUST KNOW`

| Type | How It Works | Who Does the Work | Used Between |
|------|-------------|-------------------|-------------|
| **Recursive** | "Give me the full answer" | Resolver does all work | Client → Resolver |
| **Iterative** | "I don't know, try asking X" (referral) | Each server answers what it knows | Resolver → Root → TLD → Auth |

---

## DNS Caching `MUST KNOW`

Caching happens at **every level** — browser, OS, resolver. Without caching, every lookup would traverse the full hierarchy (3+ network round trips).

### TTL (Time to Live)

Every DNS record has a TTL (seconds) — how long it can be cached.
- **Low TTL** (60s): Changes propagate fast, but more DNS queries.
- **High TTL** (86400s = 24h): Fewer queries, but changes are slow to propagate.
- **Before a migration:** Lower the TTL in advance so the old cached records expire quickly when you switch.

---

## DNS Records `MUST KNOW`

| Record | Purpose | Example |
|--------|---------|---------|
| **A** | Domain → IPv4 address | `google.com → 142.250.190.46` |
| **AAAA** | Domain → IPv6 address | `google.com → 2607:f8b0:...` |
| **CNAME** | Alias — domain → another domain | `www.example.com → example.com` |
| **MX** | Mail exchange server | `example.com → mail.example.com (priority 10)` |
| **NS** | Authoritative name servers | `google.com → ns1.google.com` |
| **TXT** | Arbitrary text (SPF, DKIM, domain verification) | `v=spf1 include:_spf.google.com ~all` |

**CNAME** cannot coexist with other records for the same name. **MX** records have priority values (lower = higher priority).

---

## DNS over HTTPS / DNS over TLS `GOOD TO KNOW`

Traditional DNS is **plaintext** over UDP — anyone on the network can see your queries.

| Method | How | Port | Key Benefit |
|--------|-----|------|-------------|
| **DoH** | DNS over HTTPS | 443 | Encrypted + looks like normal web traffic |
| **DoT** | DNS over TLS | 853 | Encrypted but uses dedicated port (easier to block) |

---
---

# PART 8 — HTTP

---

## What is HTTP? `MUST KNOW`

**HyperText Transfer Protocol** — the application-layer protocol for web communication.

- **Client-server model:** Client sends request → server sends response.
- **Stateless:** Each request is independent — server doesn't remember previous requests.
- Runs over TCP (HTTP/1.1, HTTP/2) or QUIC/UDP (HTTP/3).
- Default ports: 80 (HTTP), 443 (HTTPS).

---

## HTTP Request `MUST KNOW`

```
GET /api/users?page=1 HTTP/1.1       ← Request Line (Method, Path, Version)
Host: example.com                     ← Headers
Accept: application/json
Authorization: Bearer eyJ...
Cookie: session_id=abc123
                                      ← Empty line (separates headers from body)
                                      ← Body (empty for GET)
```

## HTTP Response `MUST KNOW`

```
HTTP/1.1 200 OK                       ← Status Line (Version, Status Code, Reason)
Content-Type: application/json        ← Headers
Content-Length: 85
Set-Cookie: session_id=xyz789
Cache-Control: max-age=3600
                                      ← Empty line
{"users": [{"id": 1, "name": "Alice"}]}  ← Body
```

---

## HTTP Methods `MUST KNOW`

| Method | Purpose | Body? | Idempotent? | Safe? |
|--------|---------|-------|-------------|-------|
| **GET** | Retrieve data | No | ✅ | ✅ |
| **POST** | Create / submit | Yes | ❌ | ❌ |
| **PUT** | Replace entire resource | Yes | ✅ | ❌ |
| **PATCH** | Partial update | Yes | ❌ | ❌ |
| **DELETE** | Delete resource | Optional | ✅ | ❌ |

### Idempotency `MUST KNOW`

Making the **same request N times** has the **same effect as making it once**.

- **GET:** Reading 10 times → same result. ✅
- **PUT:** Replacing with same data 10 times → same result. ✅
- **DELETE:** Deleting 10 times → resource is gone (later calls return 404, but state is same). ✅
- **POST:** Creating 10 times → 10 resources created. ❌ NOT idempotent.

### Safe Methods

Don't modify server state. Only **GET** (and HEAD, OPTIONS).

---

## HTTP Status Codes `MUST KNOW`

| Code | Meaning | Key Detail |
|------|---------|------------|
| **200** | OK | Standard success |
| **201** | Created | Resource created (typically POST) |
| **204** | No Content | Success, no body (typically DELETE) |
| **301** | Moved Permanently | Permanent redirect. Cached by browsers. SEO: transfers link equity. |
| **302** | Found | Temporary redirect. Don't cache. |
| **304** | Not Modified | Use cached version (ETag validation) |
| **400** | Bad Request | Client sent malformed/invalid data |
| **401** | Unauthorized | **Authentication** needed — "Who are you?" |
| **403** | Forbidden | **Authorization** failed — "I know you, but no permission." |
| **404** | Not Found | Resource doesn't exist |
| **405** | Method Not Allowed | Wrong HTTP method for this endpoint |
| **409** | Conflict | Conflicting state (e.g., duplicate resource) |
| **429** | Too Many Requests | Rate limited |
| **500** | Internal Server Error | Server crashed / unhandled exception |
| **502** | Bad Gateway | Proxy got invalid response from upstream |
| **503** | Service Unavailable | Server overloaded or in maintenance |
| **504** | Gateway Timeout | Proxy timed out waiting for upstream |

### 401 vs 403 — Critical Distinction

- **401:** "I don't know who you are. Please authenticate." (Missing/invalid credentials.)
- **403:** "I know who you are, but you can't access this." (Authenticated but not authorized.)

---

## Statelessness, Cookies, Sessions, Tokens `MUST KNOW`

### HTTP is Stateless

Each request is completely independent. The server doesn't remember previous requests.

**Why stateless?** Simplicity, scalability (any server behind a load balancer can handle any request), reliability (no state to lose on crash).

### How State is Maintained

**Cookies:**
1. Server sends `Set-Cookie: session_id=abc123` in response header.
2. Browser stores the cookie.
3. Browser sends `Cookie: session_id=abc123` with every subsequent request to that domain.
4. Server reads the cookie to identify the client.

**Sessions:**
- Server-side storage indexed by session ID.
- Session ID is carried in a cookie.
- Server looks up user state (auth info, preferences) from the session store.

**Tokens (JWT):**
- Self-contained tokens that encode user identity and claims.
- Server doesn't need server-side session storage — it validates the token's signature.
- Common in stateless APIs and microservices.

---

## HTTP Caching `GOOD TO KNOW`

### Cache-Control Header

| Directive | Meaning |
|-----------|---------|
| `max-age=3600` | Cache for 3600 seconds |
| `no-cache` | Must revalidate before using cached version |
| `no-store` | Don't cache at all |
| `public` | Any cache can store (CDN, proxy) |
| `private` | Only browser can cache |

### ETag / 304 Not Modified

1. Server sends `ETag: "abc123"` (hash of resource).
2. Next request: client sends `If-None-Match: "abc123"`.
3. Resource unchanged → server returns **304 Not Modified** (no body, saves bandwidth).
4. Resource changed → server returns **200** with new content and new ETag.

---

## Persistent Connections (Keep-Alive) `MUST KNOW`

- **HTTP/1.0:** New TCP connection for every request. Expensive.
- **HTTP/1.1:** **Persistent connections** by default — multiple requests reuse the same TCP connection.

**Why this matters:** A web page makes dozens of requests. Without keep-alive, each requires a TCP handshake (+ TLS for HTTPS) = 2-3 RTTs of overhead per request.

---
---

# PART 9 — HTTP VERSIONS

---

## HTTP/1.1 `MUST KNOW`

- **Persistent connections** (keep-alive) — default.
- **Head-of-line (HOL) blocking:** Responses must come back in order. A slow response blocks all subsequent responses on that connection.
- **Workaround:** Browsers open **6 parallel TCP connections** per domain.

## HTTP/2 `MUST KNOW`

- **Binary framing** — data encoded in binary frames, not text. More efficient to parse.
- **Multiplexing:** Multiple requests and responses over a **single TCP connection**, interleaved as frames in independent **streams**. No HTTP-level HOL blocking.
- **Header compression (HPACK):** Reduces overhead of repeated headers.
- **Server push:** Server proactively sends resources (less used in practice).

**Why HTTP/2 is faster:** Multiplexing eliminates HTTP-level HOL blocking. Single connection = 1 TCP + 1 TLS handshake (not 6). Binary is more efficient. Header compression saves bandwidth.

**BUT:** Still has **TCP-level HOL blocking** — if a TCP segment is lost, ALL streams are blocked until it's retransmitted (TCP guarantees ordered delivery).

## HTTP/3 `MUST KNOW`

- Runs over **QUIC** (built on **UDP**, not TCP).
- **No TCP-level HOL blocking:** Streams are independent at the transport level. Lost packet for stream A doesn't block streams B and C.
- **Faster connection setup:** QUIC combines transport + TLS handshake into **1 RTT** (or **0-RTT** for returning clients).
- **Connection migration:** Connections survive IP changes (e.g., switching from Wi-Fi to cellular).
- **Mandatory encryption** — TLS 1.3 built in.

### HTTP/1.1 vs HTTP/2 vs HTTP/3

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------|----------|--------|--------|
| **Transport** | TCP | TCP | QUIC (UDP) |
| **Format** | Text | Binary | Binary |
| **Multiplexing** | No | Yes (streams over 1 TCP) | Yes (independent streams) |
| **HOL Blocking** | HTTP-level | TCP-level remains | None |
| **Header Compression** | No | HPACK | QPACK |
| **Encryption** | Optional | Practically required | Mandatory (TLS 1.3) |
| **Connection Setup** | 2-3 RTT | 2-3 RTT | 1 RTT (0-RTT resumption) |
| **Connection Migration** | No | No | Yes |

---
---

# PART 10 — HTTPS + TLS

---

## Why HTTPS? `MUST KNOW`

HTTPS provides three security guarantees:

| Goal | What It Means | Without It |
|------|--------------|------------|
| **Confidentiality** | Data is encrypted — can't be read | Eavesdroppers see passwords, messages |
| **Integrity** | Data can't be modified in transit | Attackers can inject malware, alter content |
| **Authentication** | You're talking to the real server | Attackers impersonate websites |

**HTTPS = HTTP + TLS.** The HTTP protocol is identical — TLS is a layer that encrypts everything.

---

## Encryption `MUST KNOW`

### Symmetric Encryption

- **Same key** for encryption and decryption.
- **Fast** — efficient for bulk data.
- **Problem:** How do you share the key securely?
- Algorithms: AES, ChaCha20.

### Asymmetric Encryption

- **Two keys:** Public key (anyone) and Private key (only owner).
- Encrypt with public key → only private key can decrypt.
- **Slow** compared to symmetric.
- Used for key exchange and digital signatures.
- Algorithms: RSA, ECDHE.

### Why Both? (Hybrid Encryption) `MUST KNOW`

| Problem | Solution |
|---------|----------|
| Symmetric is fast but has the key distribution problem | Use asymmetric to securely exchange a symmetric key |
| Asymmetric is secure but too slow for bulk data | Once the symmetric key is shared, use it for all data |

```
TLS Handshake: Asymmetric → agree on shared secret → derive symmetric session keys
Data Transfer: Symmetric encryption with session keys (fast)
```

---

## Certificates and Certificate Authorities `MUST KNOW`

### What a Certificate Contains

- **Domain name** (e.g., `google.com`)
- **Server's public key**
- **Issuer** — which CA signed it
- **Validity period**
- **CA's digital signature** — proves the certificate is authentic

### Why Certificates Exist

Without a certificate, you can't verify that a public key actually belongs to `google.com`. An attacker could give you their own key. The certificate, signed by a trusted CA, proves the key belongs to the real server.

### Certificate Authority (CA)

A trusted third party that verifies domain ownership and signs certificates.

### Chain of Trust

```
Root CA (self-signed, trusted by OS/browser)
    ↓ signs
Intermediate CA
    ↓ signs
Server Certificate (for google.com)
```

Browser verification: traces the chain from server cert → intermediate → root. If the root is in the browser's trust store → certificate is valid.

---

## TLS Handshake `MUST KNOW`

```
Client                                          Server
  |── Client Hello ──────────────────────────── >|
  |   (TLS versions, cipher suites, random_c)    |
  |                                               |
  |<──────────────────── Server Hello ───────────|
  |   (chosen version, cipher, random_s)          |
  |<──────────────────── Certificate ─────────── |
  |   (server's public key + CA signature)        |
  |<──────────────────── Key Exchange params ──── |
  |                                               |
  | [Client verifies certificate]                 |
  |                                               |
  |── Client Key Exchange ──────────────────── > |
  |                                               |
  | [Both derive shared secret → session keys]    |
  |                                               |
  |── Finished (encrypted) ────────────────── >  |
  |<──────────────────── Finished (encrypted) ──|
  |                                               |
  |═══════ Encrypted communication begins ════════|
```

**Key steps:**
1. **Client Hello:** Supported TLS versions, cipher suites, client random.
2. **Server Hello + Certificate:** Server picks cipher, sends certificate with public key.
3. **Key Exchange:** Both sides perform Diffie-Hellman → compute the same shared secret (never transmitted).
4. **Session Keys:** Derived from shared secret + randoms. Symmetric keys for the session.
5. **Finished:** Both verify handshake succeeded with encrypted messages.

**TLS 1.2:** 2 RTTs. **TLS 1.3:** 1 RTT (0-RTT for resumption).

### Forward Secrecy

Even if the server's private key is compromised later, past sessions can't be decrypted — because Diffie-Hellman generates **ephemeral** keys per session. Mandatory in TLS 1.3.

---

## HTTPS Flow `MUST KNOW`

```
DNS Lookup (UDP:53)     → Resolve domain to IP
        ↓
TCP 3-Way Handshake     → Establish transport connection (1 RTT)
        ↓
TLS Handshake           → Authenticate server, establish session keys (1-2 RTT)
        ↓
Encrypted HTTP Request  → GET / HTTP/2 (encrypted by TLS)
        ↓
Encrypted HTTP Response → 200 OK + HTML (encrypted by TLS)
```

---
---

# PART 11 — REAL-WORLD NETWORKING COMPONENTS

---

## Proxy vs Reverse Proxy `GOOD TO KNOW`

### Forward Proxy

- Sits between **client** and the internet.
- Client sends requests to the proxy → proxy forwards to the destination.
- **Hides the client.** Server sees the proxy's IP, not the client's.
- Use cases: Content filtering, anonymity, bypassing restrictions.

### Reverse Proxy

- Sits in front of **servers**, invisible to the client.
- Client thinks it's talking to the actual server.
- **Hides the servers.** Client doesn't know about backend infrastructure.
- Use cases: Load balancing, SSL termination, caching, security.
- Examples: Nginx, HAProxy, Cloudflare.

| | Forward Proxy | Reverse Proxy |
|-|--------------|---------------|
| Position | Client side | Server side |
| Protects | Client identity | Server identity |
| Client aware? | Yes (configured) | No (transparent) |
| Use case | Filtering, anonymity | Load balancing, caching, SSL |

---

## Load Balancer `GOOD TO KNOW`

Distributes incoming requests across multiple backend servers.

**Why:** High availability (server failure doesn't cause downtime) + horizontal scaling.

**Algorithms:**
- **Round-robin:** Requests rotate through servers in order.
- **Least connections:** Send to server with fewest active connections.
- **IP hash:** Same client IP → same server (session affinity).
- **Health checks:** Periodically verify servers are alive; remove unhealthy ones.

**L4 vs L7 Load Balancing:**
- **L4 (TCP):** Routes by IP/port. Fast, but can't inspect HTTP content.
- **L7 (HTTP):** Routes by URL, headers, cookies. More flexible (e.g., `/api` to one group, `/static` to another).

---

## CDN (Content Delivery Network) `GOOD TO KNOW`

- Network of **geographically distributed edge servers** that cache content close to users.
- Serves **static content** (images, CSS, JS) from nearby edge instead of distant origin.
- **Reduces latency and origin server load.**
- How it works: First request → CDN fetches from origin, caches. Subsequent requests → served from edge.
- Examples: Cloudflare, AWS CloudFront, Akamai.

---

## Firewall `GOOD TO KNOW`

- Monitors and **filters** network traffic based on rules.
- Can block by IP address, port, protocol, or content.
- **Stateful firewall:** Tracks connection state — only allows responses to established connections.
- **WAF (Web Application Firewall):** Inspects HTTP traffic — blocks SQL injection, XSS, etc.

---

## WebSockets `GOOD TO KNOW`

### Why WebSockets Exist

Standard HTTP is request-response — client must ask to get data. WebSockets enable the server to **push** data to the client without being asked.

### How It Works

1. Client sends HTTP GET with `Upgrade: websocket` header.
2. Server responds with **101 Switching Protocols**.
3. TCP connection upgrades from HTTP to WebSocket.
4. **Full-duplex** — both sides send messages freely with tiny frame headers.

### When to Use

Chat (Slack, Discord), real-time collaboration (Google Docs), live gaming, stock tickers, monitoring dashboards — any scenario needing frequent bidirectional communication.

---

## QUIC `GOOD TO KNOW`

- Transport protocol built on **UDP** by Google.
- Provides reliability, encryption (TLS 1.3 built-in), and multiplexing with **independent streams**.
- Used by **HTTP/3**.
- Solves TCP's head-of-line blocking: lost packet for one stream doesn't block others.
- **1-RTT** connection setup (0-RTT for returning clients).
- **Connection migration** — survives IP changes.

---
---

# PART 12 — THE MOST IMPORTANT END-TO-END FLOW

---

# What Happens When You Type `https://example.com` Into a Browser? `MUST KNOW`

---

### Step 1: URL Parsing `[Application Layer]`

Browser parses the URL:
- **Scheme:** `https` → TLS, port 443.
- **Domain:** `example.com`
- **Path:** `/` (default)

Checks **HSTS list** — if domain is listed, forces HTTPS even if you typed `http://`.

---

### Step 2: DNS Lookup `[Application Layer, UDP:53]`

1. **Browser DNS cache** → miss.
2. **OS DNS cache** → miss.
3. OS queries **recursive resolver** (e.g., `8.8.8.8`) over UDP:53.
4. Resolver cache → miss.
5. Resolver → **Root** → referral to `.com` TLD.
6. Resolver → **.com TLD** → referral to `example.com` authoritative server.
7. Resolver → **Authoritative** → returns `93.184.216.34`.
8. Cached at all levels with TTL.

---

### Step 3: Determine Local vs Remote `[Network Layer]`

OS compares `93.184.216.34` with its own subnet (e.g., `192.168.1.0/24`). Not local → must route through the **default gateway**.

---

### Step 4: ARP for Default Gateway `[Data Link Layer]`

OS needs the router's MAC address to build the Ethernet frame.
- Check ARP cache for gateway IP (`192.168.1.1`).
- If not cached → ARP broadcast → router responds with its MAC.

---

### Step 5: TCP 3-Way Handshake `[Transport Layer]`

```
Client (192.168.1.10:52431)              Server (93.184.216.34:443)
  |── SYN (seq=x) ─────────────────── >|
  |<──── SYN-ACK (seq=y, ack=x+1) ────|
  |── ACK (ack=y+1) ──────────────── >|
```

Each TCP segment → wrapped in IP packet (src/dst IP) → wrapped in Ethernet frame (dst MAC = router's MAC). **At each router hop:** IP stays same, MAC gets rewritten.

**Cost:** 1 RTT.

---

### Step 6: TLS Handshake `[Security Layer]`

Client Hello → Server Hello + Certificate → Key Exchange → Session Keys → Finished.

Client verifies certificate (CA chain, domain, expiry). Both derive symmetric session keys via Diffie-Hellman.

**Cost:** 1 RTT (TLS 1.3) or 2 RTTs (TLS 1.2).

---

### Step 7: HTTP Request `[Application Layer]`

Browser sends (encrypted by TLS):
```
GET / HTTP/2
Host: example.com
User-Agent: Mozilla/5.0 ...
Accept: text/html
```

This is: encrypted by TLS → segmented by TCP → packetized by IP → framed by Ethernet → transmitted as bits.

---

### Step 8: Server-Side Processing

Request may traverse:
1. **CDN edge** — if cached, respond immediately.
2. **Load balancer** — distributes to a backend server.
3. **Reverse proxy** (Nginx) — may serve cached content.
4. **Application server** — processes request, queries database, generates response.

---

### Step 9: HTTP Response

```
HTTP/2 200 OK
Content-Type: text/html; charset=UTF-8
Content-Encoding: gzip
```

Response is encrypted by TLS, segmented by TCP (reliable delivery with ACKs), routed back through the internet.

---

### Step 10: Browser Renders

1. TCP reassembles segments in order.
2. TLS decrypts.
3. Browser parses HTTP response.
4. Discovers additional resources (CSS, JS, images).
5. Makes more HTTP requests (reusing TCP connection via keep-alive / HTTP/2 multiplexing).
6. Builds DOM → applies CSS → executes JS → renders page.

---

### Compact Version (Rapid Revision)

```
URL parsing → DNS (UDP:53) → ARP (gateway MAC) → TCP handshake (1 RTT)
→ TLS handshake (1 RTT) → HTTP request (encrypted) → Server (CDN/LB/App)
→ HTTP response (encrypted) → TCP reliable delivery → Browser renders
```

**Minimum latency before first byte:** DNS (~0-100ms) + TCP (1 RTT) + TLS (1 RTT) + server processing = **~2-3 RTTs**.

---

### Protocol Summary at Each Step

| Step | Protocol | Layer | Purpose |
|------|----------|-------|---------|
| DNS | DNS (UDP:53) | Application | Resolve domain → IP |
| ARP | ARP (broadcast) | Data Link | Resolve gateway IP → MAC |
| TCP Handshake | TCP | Transport | Establish reliable connection |
| TLS Handshake | TLS | Security | Authenticate + encrypt |
| HTTP Request | HTTP | Application | Request the web page |
| IP Routing | IP | Network | Route packets across internet |
| Ethernet | Ethernet | Data Link | Local hop-by-hop delivery |

---
---

# PART 13 — IMPORTANT COMPARISONS

---

### OSI vs TCP/IP

| Feature | OSI | TCP/IP |
|---------|-----|--------|
| Layers | 7 | 4 |
| Purpose | Conceptual reference | Practical implementation |
| Session/Presentation | Separate layers | Merged into Application |
| Used in | Interviews, academia | The actual internet |

### MAC vs IP vs Port

| | MAC | IP | Port |
|-|-----|-----|------|
| Layer | L2 | L3 | L4 |
| Size | 48 bits | 32 bits (IPv4) | 16 bits |
| Scope | Local (hop-by-hop) | End-to-end | Process on host |
| Changes per hop? | Yes | No (except NAT) | No |
| Identifies | Device on local link | Device on internet | Process/service |

### Hub vs Switch vs Router

| Device | Layer | Forwards By | Function |
|--------|-------|------------|----------|
| Hub | L1 | N/A (floods all) | Dumb repeater |
| Switch | L2 | MAC address | Selective forwarding, learns MACs |
| Router | L3 | IP address | Connects networks, routing table |

### Frame vs Packet vs Segment

| Term | Layer | Contains |
|------|-------|----------|
| Frame | L2 | Ethernet header + Packet + FCS |
| Packet | L3 | IP header + Segment/Datagram |
| Segment | L4 | TCP header + Data |
| Datagram | L4 | UDP header + Data |

### TCP vs UDP

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented | Connectionless |
| Reliability | Reliable (ACK, retransmit) | Unreliable |
| Ordering | Ordered | No ordering |
| Flow/Congestion Control | Yes / Yes | No / No |
| Header | 20-60 bytes | 8 bytes |
| Use Cases | HTTP, SSH, email | DNS, streaming, gaming |

### Flow Control vs Congestion Control

| | Flow Control | Congestion Control |
|-|---|---|
| Prevents overwhelming | **Receiver** | **Network** |
| Controlled by | Receiver (rwnd) | Sender (cwnd) |
| Mechanism | Window in TCP header | Slow start, AIMD |

### HTTP vs HTTPS

| Feature | HTTP | HTTPS |
|---------|------|-------|
| Port | 80 | 443 |
| Encryption | None | TLS |
| Security | Plaintext | Encrypted + authenticated |

### HTTP/1.1 vs HTTP/2 vs HTTP/3

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------|----------|--------|--------|
| Transport | TCP | TCP | QUIC (UDP) |
| Multiplexing | No | Yes | Yes (independent) |
| HOL Blocking | HTTP-level | TCP-level | None |
| Setup | 2-3 RTT | 2-3 RTT | 1 RTT |

### Symmetric vs Asymmetric Encryption

| | Symmetric | Asymmetric |
|-|-----------|------------|
| Keys | Same key | Public + private |
| Speed | Fast | Slow |
| Used for | Bulk data encryption | Key exchange, signatures |
| In TLS | Session keys | Handshake |

### Proxy vs Reverse Proxy

| | Forward Proxy | Reverse Proxy |
|-|---|---|
| Position | Client side | Server side |
| Protects | Client | Server |
| Use | Filtering, anonymity | Load balancing, caching |

### Public IP vs Private IP

| | Public | Private |
|-|--------|---------|
| Routable | Internet | Not internet-routable |
| Unique | Globally | Reused across networks |
| Ranges | All except private | 10.x, 172.16-31.x, 192.168.x |
| NAT needed? | No | Yes |

### ARP vs DNS

| | ARP | DNS |
|-|-----|-----|
| Resolves | IP → MAC | Domain → IP |
| Layer | L2/L3 | L7 |
| Scope | Local network only | Global (internet-wide) |
| Protocol | Broadcast/unicast | UDP:53 |

### REST vs RPC

| | REST | RPC |
|-|------|-----|
| Style | Resource-oriented (nouns: `/users/123`) | Action-oriented (verbs: `getUser(123)`) |
| Transport | HTTP (both can use it) | HTTP, HTTP/2, custom |
| Format | JSON (text) | JSON, Protobuf (binary) |
| Best for | Public APIs, web clients | Internal microservices, high-performance |

Both can use HTTP as transport, but REST organizes around resources (URLs + methods), while RPC organizes around procedures/actions.

---
---

# PART 14 — COMMON INTERVIEW QUESTIONS

---

### 1. What happens when you enter a URL in the browser?

**Ideal Answer:**
"The browser parses the URL, resolves the domain to an IP via DNS (checking browser/OS/resolver caches, then querying root→TLD→authoritative servers). It determines the destination is remote, ARPs for the default gateway's MAC, and sends packets through the router. A TCP 3-way handshake establishes a connection (1 RTT), then a TLS handshake authenticates the server and derives session keys (1 more RTT for TLS 1.3). The browser sends an encrypted HTTP request. It may pass through a CDN, load balancer, or reverse proxy before reaching the application server. The server processes the request and returns an encrypted response. TCP ensures reliable delivery. The browser decrypts, parses HTML, fetches additional resources (reusing the connection), and renders the page."

**Follow-up: Where does ARP fit in?**
"ARP resolves the gateway router's IP to its MAC address so the host can create an Ethernet frame. ARP is needed for local delivery — the packet goes to the router, which then forwards it across networks."

---

### 2. TCP vs UDP?

**Ideal Answer:**
"TCP is connection-oriented, reliable, and ordered — it guarantees delivery using ACKs and retransmission, with flow and congestion control. UDP is connectionless, unreliable, and unordered — it's faster due to no handshake or retransmission overhead. Use TCP for HTTP, file transfer, SSH. Use UDP for DNS (small queries), real-time streaming, gaming, and QUIC."

**Follow-up: Can UDP be made reliable?**
"Yes. QUIC (used by HTTP/3) builds reliability on top of UDP — it adds acknowledgments, retransmission, and flow control at the application layer, with per-stream independence that TCP can't provide."

---

### 3. Why does TCP use a 3-way handshake?

**Ideal Answer:**
"The 3-way handshake synchronizes Initial Sequence Numbers between both sides and confirms that both can send and receive. SYN establishes the client's ISN, SYN-ACK establishes the server's ISN and acknowledges the client's, and the final ACK confirms the server's ISN was received."

**Follow-up: Why isn't a 2-way handshake sufficient?**
"With only 2 steps, the server never knows if the client received its ISN. Also, old duplicate SYN packets could open phantom connections — the third ACK lets the server confirm the client is genuinely present."

---

### 4. What happens when a TCP packet is lost?

**Ideal Answer:**
"The receiver detects a gap in sequence numbers and sends duplicate ACKs. After 3 duplicate ACKs, the sender performs fast retransmit — resending the lost segment without waiting for timeout. If no ACKs arrive at all, the retransmission timeout (RTO) expires and the sender retransmits, dropping cwnd to 1 MSS and re-entering slow start. The application never sees the loss — TCP handles it transparently."

---

### 5. Flow control vs congestion control?

**Ideal Answer:**
"Flow control prevents the sender from overwhelming the **receiver** — the receiver advertises its available buffer space (rwnd) and the sender limits accordingly. Congestion control prevents the sender from overwhelming the **network** — the sender maintains a congestion window (cwnd) and adjusts based on packet loss. They're separate mechanisms solving different problems. The effective window is min(rwnd, cwnd)."

---

### 6. How does DNS work?

**Ideal Answer:**
"DNS resolves domain names to IP addresses. The client sends a recursive query to a resolver (like 8.8.8.8). The resolver sends iterative queries: first to a root server (which refers to the TLD), then to the TLD server (which refers to the authoritative), then to the authoritative server (which returns the actual IP). Results are cached at every level with TTL."

**Follow-up: Recursive vs iterative?**
"Recursive: the client expects the resolver to return the final answer. Iterative: each server responds with a referral ('try asking X'). Client→resolver is recursive; resolver→servers is iterative."

---

### 7. How does HTTPS work?

**Ideal Answer:**
"HTTPS is HTTP over TLS. After a TCP connection is established, a TLS handshake occurs: the server sends its certificate (containing its public key, signed by a CA), the client verifies it, both perform a Diffie-Hellman key exchange to derive shared symmetric session keys. All subsequent HTTP data is encrypted with these session keys. This provides confidentiality, integrity, and authentication."

**Follow-up: Why not just use asymmetric encryption for everything?**
"Asymmetric encryption is too slow for bulk data. TLS uses asymmetric only for the key exchange during the handshake, then switches to fast symmetric encryption for all application data."

---

### 8. Why do we need ARP?

**Ideal Answer:**
"Ethernet frames require MAC addresses for delivery. When a host knows the destination's IP but not its MAC, ARP resolves IP → MAC on the local network. The host broadcasts an ARP request; the target responds with its MAC. Without ARP, the host can't create the Ethernet frame."

**Follow-up: Why does a remote packet use the gateway's MAC?**
"Because MAC addresses are local — they only work within a single network segment. To reach a remote host, the packet must go through the router (default gateway). So the frame's destination MAC is the router's MAC, while the IP header still has the final destination's IP. The router strips the frame, reads the IP, and creates a new frame for the next hop."

---

### 9. What does a router do?

**Ideal Answer:**
"A router forwards packets between different networks based on IP addresses. It reads the destination IP, consults its routing table using longest prefix match, decrements TTL, and forwards the packet to the next hop — creating a new Ethernet frame with the next hop's MAC address."

---

### 10. How does NAT work?

**Ideal Answer:**
"NAT translates private IP addresses to a public IP. When a device sends a packet, the router rewrites the source IP to its public IP and assigns a unique port, recording the mapping in a NAT table. When the response comes back, the router uses the port to look up the original private IP and forwards the packet. This lets multiple devices share one public IP."

---

### 11. HTTP/1.1 vs HTTP/2 vs HTTP/3?

**Ideal Answer:**
"HTTP/1.1 uses text-based messages with one request at a time per connection, causing head-of-line blocking. HTTP/2 uses binary framing with multiplexed streams over one TCP connection, eliminating HTTP-level HOL blocking, but TCP-level HOL remains. HTTP/3 uses QUIC over UDP with truly independent streams — no HOL blocking at any level, plus 1-RTT setup and connection migration."

---

### 12. What is a socket?

**Ideal Answer:**
"A socket is a communication endpoint defined by an IP address and port number. It's the interface between the application and the transport layer. A TCP connection is identified by a 5-tuple: source IP, source port, destination IP, destination port, and protocol. This is why one server port can handle thousands of connections — each has a unique 5-tuple."

---

### 13. Why does TCP use TIME_WAIT?

**Ideal Answer:**
"Two reasons. First, to ensure the final ACK reaches the other side — if it's lost, the peer retransmits its FIN, and TIME_WAIT allows the ACK to be resent. Second, to let old duplicate packets from the closed connection expire before a new connection reuses the same port pair. Duration is 2×MSL (typically 120 seconds)."

---

### 14. How does a switch learn MAC addresses?

**Ideal Answer:**
"When a frame arrives, the switch records the source MAC and the port it arrived on in its MAC address table. Over time, it builds a complete map. When forwarding, it looks up the destination MAC — if found, it sends to that port only; if not found (unknown unicast), it floods the frame to all ports except the source."

---

### 15. Public vs private IP?

**Ideal Answer:**
"Public IPs are globally unique and internet-routable. Private IPs (10.x, 172.16-31.x, 192.168.x) are used internally and aren't routable on the internet. Private IPs exist because IPv4 doesn't have enough addresses. NAT translates private to public for internet access."

---

### 16. Proxy vs reverse proxy?

**Ideal Answer:**
"A forward proxy sits on the client side — clients send requests through it to reach the internet. It hides the client's identity (filtering, anonymity). A reverse proxy sits on the server side — clients connect to it thinking it's the server. It hides backend infrastructure and handles load balancing, SSL termination, caching."

---

### 17. What happens when a website is slow?

**Ideal Answer:**
"I'd break it down by phase: DNS resolution (slow resolver, uncached). TCP handshake (high RTT). TLS handshake (adds 1-2 RTTs). Time-to-first-byte (slow server processing — check DB queries, application logs). Content download (large payload, low bandwidth). No CDN (content served from distant origin). No connection reuse (repeated TCP+TLS handshakes). I'd use browser DevTools Network tab to measure each phase."

---

### 18. How does TCP handle congestion?

**Ideal Answer:**
"TCP starts with slow start — cwnd grows exponentially until hitting ssthresh. Then congestion avoidance — cwnd grows linearly (AIMD). If 3 duplicate ACKs occur (mild congestion), cwnd is halved and fast recovery begins. If timeout occurs (severe congestion), cwnd drops to 1 MSS and slow start restarts. Both cases set ssthresh to half the current cwnd."

---

### 19. What is the difference between 401 and 403?

**Ideal Answer:**
"401 means authentication is required — the client didn't provide valid credentials ('who are you?'). 403 means the client is authenticated but not authorized — 'I know who you are, but you don't have permission.' 401 is an identity problem; 403 is a permissions problem."

---
---

# PART 15 — COMMON MISCONCEPTIONS

---

| Misconception | Correction |
|---------------|------------|
| "TCP is slower than UDP" | TCP has more overhead, but it's not inherently "slower." For bulk transfers, TCP's flow/congestion control actually optimizes throughput. UDP's advantage is lower **latency** for real-time apps. |
| "UDP can't be reliable" | UDP itself is unreliable, but you can build reliability **on top of it** (QUIC does exactly this). |
| "A switch and a router do the same thing" | Switch forwards based on **MAC** (Layer 2, local). Router forwards based on **IP** (Layer 3, across networks). |
| "DNS sends website data" | DNS only resolves names to IP addresses. The actual website data travels over HTTP/HTTPS — a completely separate connection. |
| "ARP works across the internet" | ARP only works on the **local network** (LAN). It resolves IP→MAC for the next hop, not the final destination. |
| "HTTPS means the server is safe/trustworthy" | HTTPS means the connection is **encrypted and authenticated**. The server could still serve malware, phishing content, or have vulnerabilities. |
| "HTTP/3 uses TCP" | HTTP/3 uses **QUIC over UDP** — that's the entire point (escaping TCP's limitations). |
| "Flow control and congestion control are the same" | Flow control protects the **receiver** (rwnd). Congestion control protects the **network** (cwnd). Different problems, different mechanisms. |
| "NAT is a firewall" | NAT translates addresses (private↔public). A firewall filters traffic based on rules. NAT provides some incidental protection by hiding internal IPs, but it's not a firewall. |
| "A port is a physical thing" | A port is a **16-bit number** identifying a process/service. It's purely logical — not a physical connector. |
| "The destination MAC in a frame is always the final destination's MAC" | When sending to a remote host, the frame's destination MAC is the **router's MAC** (next hop), not the final destination's. |
| "DNS always uses UDP" | DNS primarily uses UDP, but uses **TCP** for zone transfers and responses >512 bytes. |
| "POST is always for creating resources" | POST is for submitting data. It's often used for creation, but also for form submission, triggering actions, and any non-idempotent operation. |
| "Changing a DNS record takes effect immediately" | DNS is heavily **cached**. Changes propagate only after cached entries expire (based on TTL). |
| "HTTPS encrypts the domain name" | The domain is visible in the TLS Client Hello (SNI) and DNS queries. HTTPS encrypts the path, headers, and body — not the domain. |
| "HTTP/2 solves all head-of-line blocking" | HTTP/2 solves HTTP-level HOL blocking. **TCP-level** HOL blocking remains. HTTP/3/QUIC solves both. |

---
---

# PART 16 — FINAL RAPID REVISION SHEET

---

## Top Concepts I Must Remember

1. **TCP is reliable, connection-oriented; UDP is unreliable, connectionless.**
2. **MAC changes at every hop; IP stays the same end-to-end (ignoring NAT).**
3. **TLS uses asymmetric crypto for key exchange → symmetric for data.**
4. **3-way handshake: SYN → SYN-ACK → ACK. Synchronizes sequence numbers.**
5. **Flow control = protect receiver (rwnd). Congestion control = protect network (cwnd).**
6. **Effective window = min(rwnd, cwnd).**
7. **5-tuple (src IP, src port, dst IP, dst port, protocol) = unique connection.**
8. **DNS is a separate UDP query before TCP/TLS/HTTP.**
9. **HTTP is stateless. Cookies/sessions maintain state externally.**
10. **HTTPS = HTTP + TLS. Confidentiality + integrity + authentication.**
11. **Router = L3 (IP). Switch = L2 (MAC). Hub = L1 (dumb).**
12. **Longest prefix match: most specific route wins.**
13. **ARP resolves IP → MAC on the local network. Broadcast-based.**
14. **Remote traffic: ARP resolves the gateway's MAC, not the destination's.**
15. **Loss detection: 3 dup ACKs → fast retransmit (cwnd halved). Timeout → cwnd=1, slow start.**
16. **4-way termination: each direction closed independently (full-duplex).**
17. **TIME_WAIT (2×MSL): ensures final ACK arrives + old packets expire.**
18. **HTTP/2: multiplexed streams. HTTP/3: QUIC/UDP, no HOL blocking.**
19. **Certificate binds domain + public key, signed by CA. Chain of trust.**
20. **NAT/PAT: multiple private IPs share one public IP via port mapping.**

---

## Important Protocols Table

| Protocol | Purpose | Layer | Key Interview Point |
|----------|---------|-------|---------------------|
| **Ethernet** | LAN framing, local delivery | L2 | MAC addressing, MTU=1500, CRC |
| **ARP** | IP → MAC resolution | L2/L3 | Broadcast request, unicast reply, local only |
| **IP** | Logical addressing, routing | L3 | 32-bit (v4), TTL, routing via longest prefix match |
| **ICMP** | Error reporting, diagnostics | L3 | Ping (echo), traceroute (TTL exceeded) |
| **TCP** | Reliable, ordered transport | L4 | 3-way handshake, flow/congestion control, retransmission |
| **UDP** | Fast, unreliable transport | L4 | 8-byte header, no handshake, DNS/streaming/QUIC |
| **DNS** | Domain → IP resolution | L7 | Hierarchical, UDP:53, caching with TTL |
| **HTTP** | Web communication | L7 | Stateless, methods, status codes, request/response |
| **HTTPS/TLS** | Encrypted web communication | L7/Security | Certificates, handshake, session keys |
| **QUIC** | Reliable transport over UDP | L4 | HTTP/3, per-stream reliability, 0-RTT |

---

## Important Port Numbers

| Port | Service |
|------|---------|
| 22 | SSH |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |

---

## Important Flows (Compact)

### ARP
Host checks cache → miss → broadcast "Who has X?" → target replies with MAC → cache it → send frame.

### DNS
Browser cache → OS cache → resolver cache → resolver queries root → TLD → authoritative → IP returned → cached with TTL.

### TCP Handshake
SYN(seq=x) → SYN-ACK(seq=y, ack=x+1) → ACK(ack=y+1). **1 RTT.**

### TCP Termination
FIN → ACK → FIN → ACK. Initiator enters TIME_WAIT (2×MSL).

### TLS Handshake
Client Hello → Server Hello + Certificate → Key Exchange (DH) → Session Keys → Finished. **1 RTT (TLS 1.3).**

### HTTP Request
`METHOD /path HTTP/1.1` + headers + body → Server processes → `HTTP/1.1 STATUS Reason` + headers + body.

### Complete HTTPS Request
DNS (UDP:53) → TCP handshake (1 RTT) → TLS handshake (1 RTT) → Encrypted HTTP → Response → Render.

---

## 50 Highest-Value Interview Questions (Concise Answers)

1. **What is the OSI model?** — 7-layer reference: Physical, Data Link, Network, Transport, Session, Presentation, Application. TCP/IP (4 layers) is what the internet uses.
2. **What is encapsulation?** — Each layer adds its header: Data → Segment → Packet → Frame → Bits.
3. **MAC vs IP vs Port?** — MAC (L2, local, 48-bit). IP (L3, end-to-end, 32-bit). Port (L4, process, 16-bit).
4. **Hub vs Switch vs Router?** — Hub: L1, floods all. Switch: L2, forwards by MAC. Router: L3, forwards by IP.
5. **What is the 5-tuple?** — (src IP, src port, dst IP, dst port, protocol). Uniquely identifies a connection.
6. **What is ARP?** — Resolves IP → MAC on local network. Broadcast request, unicast reply.
7. **How does a switch learn MACs?** — Records source MAC + port on frame arrival. Builds table over time.
8. **Collision domain vs broadcast domain?** — Switch separates collision domains. Router separates broadcast domains.
9. **What is subnetting?** — Dividing networks into smaller subnets. CIDR: /24 = 254 hosts, /27 = 30 hosts.
10. **What is NAT?** — Translates private IPs to public. PAT uses ports to multiplex many devices through one public IP.
11. **What is TTL?** — Decremented at each hop. At 0 → packet dropped + ICMP Time Exceeded. Prevents loops.
12. **What is TCP?** — Connection-oriented, reliable, ordered transport. 3-way handshake, ACKs, retransmission.
13. **What is UDP?** — Connectionless, unreliable, fast. 8-byte header. DNS, streaming, gaming, QUIC.
14. **TCP vs UDP?** — TCP: reliable/ordered/slow. UDP: unreliable/fast. Use TCP for HTTP; UDP for real-time.
15. **Explain the 3-way handshake.** — SYN(seq=x) → SYN-ACK(seq=y, ack=x+1) → ACK(ack=y+1). 1 RTT.
16. **Why 3-way, not 2-way?** — Both sides must confirm each other's ISN. 2-way can't confirm server's ISN.
17. **How does TCP detect loss?** — Timeout (severe → cwnd=1) or 3 dup ACKs (fast retransmit → cwnd halved).
18. **What is flow control?** — Receiver advertises rwnd. Prevents sender from overwhelming receiver.
19. **What is congestion control?** — Sender adjusts cwnd. Slow start → congestion avoidance → AIMD. Prevents network congestion.
20. **Flow control vs congestion control?** — Flow: receiver limit (rwnd). Congestion: network limit (cwnd). Different problems.
21. **Explain TCP termination.** — 4-way: FIN → ACK → FIN → ACK. Full-duplex, each direction closes independently.
22. **Why TIME_WAIT?** — Ensure final ACK arrives + let old packets expire. 2×MSL.
23. **What is DNS?** — Resolves domain names to IPs. Hierarchical: root → TLD → authoritative. UDP:53.
24. **How does DNS resolution work?** — Browser/OS cache → resolver → root → TLD → authoritative → IP. Cached with TTL.
25. **Recursive vs iterative DNS?** — Recursive: resolver returns final answer. Iterative: each server gives referral.
26. **Important DNS records?** — A (IPv4), AAAA (IPv6), CNAME (alias), MX (mail), NS (nameserver), TXT (verification).
27. **Why does DNS use UDP?** — Queries are small. No connection overhead. TCP for zone transfers and large responses.
28. **What is HTTP?** — Application-layer request/response protocol. Stateless. Methods + status codes.
29. **HTTP methods + idempotency?** — GET/PUT/DELETE: idempotent. POST: NOT idempotent. GET: safe.
30. **401 vs 403?** — 401: not authenticated. 403: authenticated but not authorized.
31. **Why is HTTP stateless?** — Simplicity + scalability. Any server handles any request. State via cookies/sessions.
32. **How do cookies work?** — Server: Set-Cookie → Browser stores → Browser: Cookie header → Server identifies client.
33. **HTTP/1.1 vs HTTP/2?** — HTTP/2: binary, multiplexed streams, HPACK. Eliminates HTTP-level HOL blocking.
34. **What is HOL blocking?** — Slow response blocks others. HTTP/2 fixes at HTTP level. TCP-level remains. HTTP/3 fixes both.
35. **HTTP/3 and QUIC?** — QUIC over UDP. Independent streams, no HOL blocking, 1-RTT, connection migration.
36. **HTTP vs HTTPS?** — HTTPS = HTTP + TLS. Encrypts data, verifies server identity, ensures integrity.
37. **Symmetric vs asymmetric encryption?** — Symmetric: same key, fast (AES). Asymmetric: public/private, slow (RSA). TLS uses both.
38. **How does TLS work?** — Handshake: hello → certificate → key exchange (DH) → session keys → encrypted data.
39. **What is a certificate?** — Binds domain + public key, signed by CA. Browser verifies chain of trust.
40. **What happens when you type a URL?** — DNS → ARP → TCP → TLS → HTTP → server → response → render.
41. **What is a socket?** — IP + port. Communication endpoint. TCP connection = unique 5-tuple.
42. **How can a server handle many clients on one port?** — 5-tuple. Same server port, different client IP:port combos.
43. **What is a CDN?** — Geographically distributed cache. Serves content from nearby edge. Reduces latency.
44. **Proxy vs reverse proxy?** — Forward: client side, hides client. Reverse: server side, hides servers.
45. **What is a load balancer?** — Distributes requests across servers. HA + scaling. Round-robin, least connections.
46. **Public vs private IP?** — Public: internet-routable. Private: internal (10.x, 172.16-31.x, 192.168.x). NAT bridges them.
47. **How does NAT work?** — Rewrites src IP to public + unique port. Tracks in NAT table. Reverse on response.
48. **What is longest prefix match?** — Router picks most specific route. /24 beats /16 beats /0 (default).
49. **Ping works but HTTP doesn't. Why?** — Different protocols/ports. Firewall may block TCP:80/443 but allow ICMP.
50. **Website is slow. How to troubleshoot?** — Check DNS time, TCP time, TLS time, TTFB, content download. DevTools Network tab.

---

## One-Day-Before-Interview Revision Checklist

- [ ] OSI model (7 layers) and TCP/IP model (4 layers) — what each does
- [ ] Encapsulation: Data → Segment → Packet → Frame → Bits
- [ ] MAC vs IP vs Port — scope and purpose of each
- [ ] Switch (L2, MAC) vs Router (L3, IP) — what they forward and why
- [ ] ARP — full flow, why gateway MAC is used for remote hosts
- [ ] Subnetting — calculate network, broadcast, usable hosts from CIDR
- [ ] NAT/PAT — how private → public translation works
- [ ] **TCP 3-way handshake** — draw it with sequence numbers
- [ ] TCP reliability — seq/ACK, retransmission, fast retransmit
- [ ] **Flow control vs congestion control** — explain the difference clearly
- [ ] Slow start → congestion avoidance → fast recovery → timeout
- [ ] **TCP 4-way termination** — why 4 steps, TIME_WAIT
- [ ] TCP vs UDP — differences and when to use each
- [ ] DNS resolution — full recursive/iterative flow
- [ ] DNS records — A, AAAA, CNAME, MX, NS, TXT
- [ ] HTTP methods — GET, POST, PUT, PATCH, DELETE + idempotency
- [ ] HTTP status codes — 200, 201, 301, 302, 400, 401, 403, 404, 500, 502, 503
- [ ] HTTP statelessness — cookies, sessions, tokens
- [ ] HTTP/1.1 vs HTTP/2 vs HTTP/3 — multiplexing, HOL blocking, QUIC
- [ ] Symmetric vs asymmetric encryption — why TLS uses both
- [ ] TLS handshake — certificate, DH key exchange, session keys
- [ ] **"What happens when you type https://google.com?"** — complete flow
- [ ] Proxy vs reverse proxy vs CDN vs load balancer
- [ ] Sockets — 5-tuple, bind/listen/accept/connect
- [ ] Common misconceptions (MAC changes per hop, ARP is local, etc.)

---

> **You're ready. Go get that offer.** 🚀
