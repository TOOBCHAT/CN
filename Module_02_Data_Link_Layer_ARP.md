# Module 2 — Data Link Layer + ARP

---

## 1. Data Link Layer — Overview `MUST KNOW`

**What it is:** Layer 2 of the OSI model. Responsible for **node-to-node delivery** of frames on a local network segment.

**Why it exists:** The Network layer (IP) knows the final destination, but it can't physically deliver data to the next device on the wire. The Data Link layer handles **local delivery** — getting the frame from one device to the next device on the same network segment using MAC addresses.

**Key responsibilities:**
- **Framing:** Wrapping packets into frames with headers/trailers.
- **Physical addressing:** Using MAC addresses for local delivery.
- **Error detection:** CRC (Cyclic Redundancy Check) in the frame trailer to detect corruption.
- **Media access control:** Deciding who gets to transmit on a shared medium.

---

## 2. Ethernet `MUST KNOW`

**What it is:** The dominant Layer 2 protocol for wired LANs. When you hear "Data Link Layer" in interviews, they almost always mean Ethernet.

**Why it matters:** Virtually every wired network uses Ethernet. It defines how frames are structured and how devices share the physical medium.

### Ethernet Frame Structure

```
| Preamble | Dest MAC | Src MAC | EtherType | Payload (46-1500 bytes) | FCS |
| 8 bytes  | 6 bytes  | 6 bytes | 2 bytes   | IP Packet               | 4 bytes |
```

**Key fields:**
| Field | Size | Purpose |
|-------|------|---------|
| **Destination MAC** | 6 bytes | MAC address of the next-hop device |
| **Source MAC** | 6 bytes | MAC address of the sending device |
| **EtherType** | 2 bytes | Identifies the upper-layer protocol (e.g., `0x0800` = IPv4, `0x0806` = ARP) |
| **Payload** | 46–1500 bytes | The IP packet (or other L3 data) |
| **FCS** (Frame Check Sequence) | 4 bytes | CRC for error detection — if the frame is corrupted, it's silently dropped |

**Interview points:**
- MTU (Maximum Transmission Unit) for Ethernet = **1500 bytes** (the max payload size). This is why IP fragmentation exists — if a packet exceeds the MTU, it must be broken up.
- Ethernet does error **detection** (CRC), not error **correction**. If a frame is corrupted, it's dropped. Reliability is TCP's job.

---

## 3. MAC Addresses — Deeper Look `MUST KNOW`

- **48 bits** (6 bytes), written as `AA:BB:CC:DD:EE:FF`.
- First 3 bytes = **OUI** (Organizationally Unique Identifier) — identifies the manufacturer.
- Last 3 bytes = device-specific identifier.
- **Broadcast MAC:** `FF:FF:FF:FF:FF:FF` — a frame sent to this address is received by ALL devices on the LAN.
- MAC addresses are (in theory) globally unique and burned into the NIC, though they can be spoofed in software.

**Interview point:** MAC addresses are **flat** (no hierarchy), unlike IP addresses which have a network/host structure. This is exactly why you can't route across the internet using MAC alone — there's no way to aggregate/summarize routes.

---

## 4. Switches `MUST KNOW`

**What it is:** A Layer 2 device that forwards frames based on MAC addresses.

**Why it exists:** To efficiently deliver frames only to the port where the destination device is connected, instead of flooding to everyone (which is what a hub does).

### How a Switch Learns MAC Addresses

A switch maintains a **MAC address table** (also called CAM table) that maps MAC addresses to switch ports. It learns this table dynamically:

**Step-by-step process:**

1. **A frame arrives on port 1** from device A (MAC: `AA:AA:AA:AA:AA:AA`).
2. The switch reads the **source MAC** and records: `AA:AA:AA:AA:AA:AA → Port 1`.
3. The switch reads the **destination MAC** and looks it up in the table.
4. **Three possible outcomes:**

| Scenario | What Happens |
|----------|-------------|
| **Destination MAC found in table** | Forward the frame **only** to that specific port (unicast forwarding) |
| **Destination MAC NOT in table** | **Flood** the frame out ALL ports except the one it arrived on |
| **Destination MAC is broadcast** (`FF:FF:FF:FF:FF:FF`) | **Flood** to all ports except the source port |

5. When device B replies, the switch learns B's MAC on that port too. Future frames to B go directly to B's port — no flooding needed.

**Key points:**
- Switch learns from **source MAC** (incoming direction).
- Switch forwards based on **destination MAC** (outgoing direction).
- MAC table entries have a **timeout** (typically 300 seconds). If a device is silent for too long, its entry is removed.
- This is called **transparent bridging** — the switch is invisible to the connected devices.

### Frame Forwarding Summary

```
Frame arrives → Learn Source MAC/Port → Look up Destination MAC
                                              ↓
                     ┌────────────────────────────────────────┐
                     │ Found?          Not Found?   Broadcast?│
                     │ Forward to      Flood to     Flood to  │
                     │ specific port   all ports    all ports  │
                     └────────────────────────────────────────┘
```

---

## 5. Hub vs Switch `MUST KNOW`

| Feature | Hub | Switch |
|---------|-----|--------|
| Layer | Layer 1 (Physical) | Layer 2 (Data Link) |
| Intelligence | None — repeats signal | Smart — learns MACs, forwards selectively |
| Forwarding | Floods to ALL ports | Forwards to specific port (or floods if unknown) |
| Collision domain | One shared collision domain | Each port is its own collision domain |
| Bandwidth | Shared among all devices | Dedicated per port |
| Modern usage | Obsolete | Standard in all networks |

**Interview answer:** "A hub is a dumb Layer 1 device that simply repeats incoming signals to all ports, creating one big collision domain. A switch is a smart Layer 2 device that learns MAC addresses and forwards frames only to the correct port, creating a separate collision domain per port. Switches are far more efficient and have completely replaced hubs."

---

## 6. Switch vs Router `MUST KNOW`

| Feature | Switch | Router |
|---------|--------|--------|
| Layer | Layer 2 | Layer 3 |
| Addresses used | MAC addresses | IP addresses |
| Function | Connects devices within a LAN | Connects different networks |
| Forwarding decision | MAC address table lookup | Routing table lookup |
| Broadcast handling | Forwards broadcasts within LAN | **Blocks** broadcasts (boundary) |
| Domain | Operates within a broadcast domain | Separates broadcast domains |

**Interview answer:** "A switch operates at Layer 2 and forwards frames based on MAC addresses within a single LAN. A router operates at Layer 3 and forwards packets based on IP addresses between different networks. Routers are broadcast boundaries — broadcasts don't cross them. In a typical network, switches connect devices within a LAN, and a router connects that LAN to the internet or other networks."

---

## 7. Collision Domain vs Broadcast Domain `MUST KNOW`

### Collision Domain
- The set of devices that can **collide** when transmitting simultaneously.
- A **hub** creates one collision domain for all ports.
- A **switch** creates a separate collision domain per port (collisions are eliminated on each full-duplex link).

### Broadcast Domain
- The set of devices that receive a **broadcast** frame.
- All ports on a **switch** are in the same broadcast domain (by default).
- A **router** breaks/separates broadcast domains.

**Summary:**

| Device | Collision Domains | Broadcast Domains |
|--------|------------------|------------------|
| Hub | 1 (all ports share) | 1 |
| Switch | 1 per port | 1 (all ports share by default) |
| Router | 1 per interface | 1 per interface (separates them) |

**Interview answer:** "A collision domain is the set of devices where simultaneous transmission can cause a collision. A switch eliminates collisions by giving each port its own collision domain. A broadcast domain is the set of devices that receive broadcast frames. All switch ports are in the same broadcast domain by default. Routers separate broadcast domains — broadcasts don't cross router boundaries."

---

## 8. ARP — Address Resolution Protocol `MUST KNOW`

**What it is:** A protocol that maps an **IP address** to a **MAC address** on the local network.

**Why it exists:** When your computer wants to send a frame, it needs the destination's MAC address. But it only knows the destination's IP address (from DNS, routing, etc.). ARP bridges this gap.

**How it works:**

### ARP Request (Broadcast)
1. Device A wants to send a packet to `192.168.1.5` on the same LAN.
2. A knows the destination IP but NOT the MAC address.
3. A sends an **ARP Request** as a **broadcast** (`FF:FF:FF:FF:FF:FF`):
   - "Who has IP `192.168.1.5`? Tell `192.168.1.10` (my IP)."
4. Every device on the LAN receives this broadcast.

### ARP Reply (Unicast)
5. Device B (which has IP `192.168.1.5`) responds with an **ARP Reply** (unicast, directly to A):
   - "I am `192.168.1.5`. My MAC is `BB:BB:BB:BB:BB:BB`."
6. All other devices ignore the ARP Request.

### ARP Cache
7. Device A stores this mapping in its **ARP cache/table**: `192.168.1.5 → BB:BB:BB:BB:BB:BB`.
8. Future packets to `192.168.1.5` use the cached MAC — no ARP needed.
9. ARP cache entries have a **TTL** (typically 20–60 seconds on Linux, longer on Windows). They expire and must be refreshed.

```
Device A                                          Device B
   |                                                  |
   |--- ARP Request (broadcast) --------------------->|  "Who has 192.168.1.5?"
   |                                                  |
   |<-- ARP Reply (unicast) --------------------------|  "I'm 192.168.1.5, MAC=BB:BB:..."
   |                                                  |
   [Stores in ARP cache]                              |
   |                                                  |
   |--- Data Frame (unicast, dst=BB:BB:...) --------->|
```

---

## 9. Communication Within the Same LAN `MUST KNOW`

**Scenario:** Device A (`192.168.1.10`, MAC: `AA:..`) wants to send data to Device B (`192.168.1.5`, MAC: unknown) — both on the same LAN.

**Complete flow:**

1. **Application layer:** A generates data (e.g., HTTP request).
2. **Transport layer:** TCP wraps it in a segment (src port, dst port).
3. **Network layer:** IP wraps it in a packet (src IP: `192.168.1.10`, dst IP: `192.168.1.5`).
4. **Routing decision:** A compares B's IP with its own subnet. Same subnet → deliver locally. A needs B's MAC address.
5. **ARP check:** A checks its ARP cache for `192.168.1.5`.
   - **Cache hit:** Use the cached MAC. Skip to step 8.
   - **Cache miss:** Perform ARP (steps 6–7).
6. **ARP Request:** A broadcasts "Who has `192.168.1.5`?"
7. **ARP Reply:** B responds with its MAC. A caches it.
8. **Data Link layer:** A creates an Ethernet frame: `Src MAC = AA:..`, `Dst MAC = BB:..`.
9. **Switch:** Receives the frame, learns A's MAC on its port, looks up B's MAC, forwards to B's port.
10. **B receives:** Decapsulates frame → packet → segment → data.

---

## 10. Communication Outside the Local Network `MUST KNOW`

**Scenario:** Device A (`192.168.1.10`) wants to reach `google.com` (`142.250.x.x`) — different network.

**Complete flow:**

1. **Routing decision:** A compares the destination IP (`142.250.x.x`) with its own subnet (`192.168.1.0/24`). Different subnet → **must send to default gateway** (router, e.g., `192.168.1.1`).
2. **ARP for the gateway:** A needs the MAC address of `192.168.1.1` (the router). If not cached, A does ARP for `192.168.1.1`.
3. **Frame creation:** A creates a frame:
   - **Dst MAC = router's MAC** (NOT Google's MAC — A doesn't know it and never will).
   - **Dst IP = `142.250.x.x`** (Google's IP — stays the same end-to-end).
4. **Router receives the frame:** Strips L2 header, reads the IP header, consults routing table, determines next hop.
5. **Router creates a new frame** with the next hop's MAC address and forwards it.
6. This continues hop by hop until the packet reaches Google's server.

> [!IMPORTANT]
> **The destination MAC in the frame is always the next hop's MAC, not the final destination's MAC.**
> The destination IP in the packet is always the final destination's IP.
> This is the fundamental difference between L2 (hop-by-hop) and L3 (end-to-end) addressing.

---

## 11. VLANs — Interview Level `GOOD TO KNOW`

**What it is:** A **Virtual LAN** — a way to logically segment a single physical switch into multiple isolated broadcast domains.

**Why it exists:** Without VLANs, every port on a switch is in the same broadcast domain. In a large office, you'd want to separate HR traffic from Engineering traffic for security and performance — even if they're on the same physical switch.

**How it works:**
- Each switch port is assigned to a VLAN (e.g., VLAN 10 for HR, VLAN 20 for Engineering).
- Devices in VLAN 10 can only communicate with other VLAN 10 devices at Layer 2.
- To communicate across VLANs, traffic must go through a **router** (inter-VLAN routing).
- **Trunk ports** carry traffic for multiple VLANs between switches (tagged with VLAN IDs using 802.1Q).

**Interview answer:** "A VLAN logically segments a physical switch into separate broadcast domains. Devices in different VLANs can't communicate at Layer 2 — they need a router for inter-VLAN routing. VLANs improve security and reduce broadcast traffic. Trunk ports carry traffic for multiple VLANs between switches using 802.1Q tagging."

---

## Interview Questions + Answers

---

**Q1: What does the Data Link Layer do?**

**Ideal Answer:**
"The Data Link Layer handles node-to-node delivery of frames on a local network segment. It uses MAC addresses for physical addressing, wraps IP packets into Ethernet frames, and performs error detection using CRC. It's responsible for getting data from one device to the next device on the same link — switches operate at this layer."

---

**Q2: What is a MAC address and how is it different from an IP address?**

**Ideal Answer:**
"A MAC address is a 48-bit hardware address burned into the NIC, used for local frame delivery within a LAN. An IP address is a 32-bit logical address used for routing across networks. MAC addresses are flat with no hierarchy, while IP addresses have a network/host structure that enables scalable routing. MAC changes at every hop; IP stays the same end-to-end."

---

**Q3: How does a switch work?**

**Ideal Answer:**
"A switch operates at Layer 2 and maintains a MAC address table mapping MAC addresses to ports. When a frame arrives, the switch learns the source MAC on that port. It then looks up the destination MAC in its table — if found, it forwards the frame only to that port; if not found, it floods the frame out all ports except the source. Over time, the switch learns where all devices are and can forward efficiently without flooding."

> **Follow-up Q: What happens when a switch receives a broadcast frame?**
>
> **Ideal Answer:** "The switch floods the broadcast frame to all ports except the one it arrived on. This is expected behavior — broadcasts like ARP requests need to reach all devices in the broadcast domain."

> **Follow-up Q: When does a switch flood a unicast frame?**
>
> **Ideal Answer:** "When the destination MAC address is not in the switch's MAC table — this is called an 'unknown unicast.' The switch has no choice but to flood because it doesn't know which port the destination is on. Once the destination responds, the switch learns its MAC and port, and future frames are forwarded directly."

---

**Q4: What is the difference between a hub and a switch?**

**Ideal Answer:**
"A hub is a Layer 1 device that blindly repeats signals to all ports — creating one shared collision domain. A switch is a Layer 2 device that learns MAC addresses and forwards frames only to the correct port, giving each port its own collision domain. Switches are far more efficient and have completely replaced hubs in modern networks."

---

**Q5: What is the difference between a switch and a router?**

**Ideal Answer:**
"A switch operates at Layer 2, forwards frames using MAC addresses, and connects devices within a LAN. A router operates at Layer 3, forwards packets using IP addresses, and connects different networks. A key difference is that routers are broadcast boundaries — broadcasts stay within a LAN and don't cross routers. In a typical setup, switches handle local traffic and a router connects the LAN to the internet."

---

**Q6: Explain collision domain vs broadcast domain.**

**Ideal Answer:**
"A collision domain is the set of devices where simultaneous transmissions can collide — hubs create one shared collision domain, while switches give each port its own. A broadcast domain is the set of devices that receive a broadcast frame — all ports on a switch share one broadcast domain by default. Routers separate broadcast domains. So switches fix the collision problem, and routers fix the broadcast problem."

---

**Q7: What is ARP and how does it work?**

**Ideal Answer:**
"ARP — Address Resolution Protocol — resolves IP addresses to MAC addresses on the local network. When a device needs to send a frame but only knows the destination IP, it broadcasts an ARP request: 'Who has this IP?' The device with that IP responds with a unicast ARP reply containing its MAC address. The sender caches this mapping in its ARP table for future use, with entries expiring after a TTL."

> **Follow-up Q: Why is ARP necessary?**
>
> **Ideal Answer:** "Because Ethernet frames require MAC addresses for delivery, but applications and routing work with IP addresses. ARP bridges this gap — it translates the logical IP address into the physical MAC address needed to actually put a frame on the wire."

> **Follow-up Q: Is ARP used when the destination is on a different network?**
>
> **Ideal Answer:** "Yes, but the ARP is for the **default gateway's** IP, not the final destination's IP. The host knows it can't reach the destination directly, so it ARPs for the router's MAC address and sends the frame to the router. The router then handles forwarding across networks."

---

**Q8: What happens when Device A wants to communicate with Device B on the same LAN?**

**Ideal Answer:**
"A checks if B's IP is in the same subnet. Since it is, A checks its ARP cache for B's MAC. If not cached, A broadcasts an ARP request asking for B's MAC. B responds with its MAC via unicast ARP reply. A caches the mapping, creates an Ethernet frame with B's MAC as the destination, and sends it. The switch receives the frame, looks up B's MAC in its table, and forwards the frame to B's port."

---

**Q9: What happens when the destination is on a different network?**

**Ideal Answer:**
"The host compares the destination IP against its subnet mask and determines it's on a different network. It then needs to send the packet to its default gateway (router). The host ARPs for the router's MAC address (if not cached), creates a frame with the router's MAC as the destination but keeps the original destination IP in the IP header. The router receives the frame, strips the L2 header, reads the destination IP, consults its routing table, and forwards the packet with a new frame addressed to the next hop. This repeats until the packet reaches the destination."

---

**Q10: What is a VLAN?**

**Ideal Answer:**
"A VLAN is a virtual LAN that logically divides a physical switch into separate broadcast domains. Ports assigned to different VLANs can't communicate at Layer 2 — they need a router for inter-VLAN routing. VLANs improve security by isolating traffic and reduce unnecessary broadcast traffic. Traffic between switches for multiple VLANs uses trunk ports with 802.1Q tagging."

---

**Q11: What is the Ethernet MTU and why does it matter?**

**Ideal Answer:**
"The Ethernet MTU is 1500 bytes — the maximum payload a single Ethernet frame can carry. If an IP packet exceeds 1500 bytes, it must be fragmented into multiple packets or the sender must reduce the packet size. This is important because fragmentation adds overhead and reduces performance. Modern systems typically use Path MTU Discovery to find the largest packet size that can traverse the entire path without fragmentation."

---

**Q12: Does Ethernet provide reliable delivery?**

**Ideal Answer:**
"No. Ethernet performs error detection using CRC in the Frame Check Sequence — if a frame is corrupted, it's silently dropped. There's no retransmission at Layer 2. Reliable delivery is handled by TCP at the Transport layer. This is by design — keeping Layer 2 simple and fast, and letting higher layers handle reliability if needed."

---

### Common Mistakes

| Mistake | Correction |
|---------|------------|
| "Switches use IP addresses for forwarding" | Switches use **MAC addresses**. Routers use IP addresses. (Layer 2 vs Layer 3) |
| "ARP resolves domain names to IPs" | That's **DNS**. ARP resolves **IP addresses to MAC addresses**. |
| "ARP is used to find the destination server's MAC across the internet" | ARP is **local only**. When sending to a remote server, ARP resolves the **default gateway's MAC**, not the server's. |
| "A switch forwards broadcasts to only one port" | Switches **flood** broadcasts to all ports (except the source). That's how broadcast works. |
| "A router and switch do the same thing" | A switch connects devices within a LAN (L2, MAC). A router connects different networks (L3, IP) and blocks broadcasts. |
| "Collision domain and broadcast domain are the same" | Collision domain = where collisions can happen (per port on a switch). Broadcast domain = where broadcasts reach (all switch ports, bounded by routers). |

---

### Interview Takeaways — Module 2

1. **Data Link Layer = local delivery using MAC addresses.** Switches operate here.
2. **Ethernet frame:** Dst MAC + Src MAC + EtherType + Payload (≤1500 bytes MTU) + FCS (CRC).
3. **Switches learn source MACs**, forward based on destination MACs, and flood when the destination is unknown.
4. **Hub = dumb repeater (L1). Switch = smart forwarder (L2). Router = inter-network forwarder (L3).**
5. **Switch: separate collision domains per port, one broadcast domain. Router: separates broadcast domains.**
6. **ARP maps IP → MAC on the local network.** ARP request = broadcast, ARP reply = unicast.
7. **Same LAN:** ARP for destination's MAC → send frame directly.
8. **Different network:** ARP for **default gateway's MAC** → send frame to router → router forwards.
9. **Frame's destination MAC = next hop. Packet's destination IP = final destination.** Always.
10. **VLAN = logical segmentation** of a switch into separate broadcast domains. Inter-VLAN requires a router.
