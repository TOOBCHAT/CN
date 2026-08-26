# Module 3 — IP Addressing + Subnetting + Routing

> **HIGH PRIORITY MODULE**

---

## 1. IPv4 Addressing `MUST KNOW`

### What It Is
- A **32-bit** logical address assigned to every device on a network.
- Written in **dotted decimal**: `192.168.1.10` (4 octets, each 0–255).
- Binary: `11000000.10101000.00000001.00001010`

### Network Portion vs Host Portion
Every IP address has two parts:
- **Network portion:** Identifies which network the device is on.
- **Host portion:** Identifies which specific device within that network.

The **subnet mask** determines where the split happens.

```
IP:          192.168.1.10     = 11000000.10101000.00000001.00001010
Subnet Mask: 255.255.255.0   = 11111111.11111111.11111111.00000000
                                |-------- Network --------| Host |
```

### Subnet Mask
- A 32-bit value where **1s = network bits**, **0s = host bits**.
- `255.255.255.0` = `/24` = first 24 bits are network, last 8 are host.
- Bitwise AND of IP and subnet mask = **network address**.

```
192.168.1.10  AND  255.255.255.0  =  192.168.1.0  (network address)
```

### Default Gateway
- The **router's IP address** on your local network.
- When you send a packet to a destination outside your subnet, it goes to the default gateway.
- Every host must know its default gateway to reach other networks.
- Example: Your IP is `192.168.1.10`, gateway is `192.168.1.1` (the router).

### Private vs Public IP `MUST KNOW`

| Type | Ranges | Usage |
|------|--------|-------|
| **Private** | `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` | Internal networks (home, office). NOT routable on the internet. |
| **Public** | Everything else (assigned by ISPs) | Routable on the internet. Globally unique. |

**Why private IPs exist:** IPv4 only has ~4.3 billion addresses. NAT allows many private IPs to share one public IP.

### Special Addresses

| Address | Purpose |
|---------|---------|
| `127.0.0.1` | **Loopback** — refers to yourself. Used for testing. `ping 127.0.0.1` tests your own TCP/IP stack. |
| `169.254.x.x` | **APIPA** — auto-assigned when DHCP fails. If you see this, DHCP is broken. |
| `0.0.0.0` | Represents "all interfaces" or "default route" depending on context. |
| `255.255.255.255` | Limited broadcast — broadcast to everyone on the local network. |

---

## 2. Classful Addressing `GOOD TO KNOW`

> Context for why CIDR exists. Know enough for a quick interview answer.

### What It Was
IPv4's **original addressing scheme** (before CIDR) divided the entire IP space into fixed classes based on the first few bits of the address.

| Class | First Bits | First Octet Range | Default Mask | Purpose | Example |
|-------|-----------|-------------------|-------------|---------|---------|
| **A** | `0xxxxxxx` | 1–126 | /8 | Huge networks (16M hosts) | 10.x.x.x |
| **B** | `10xxxxxx` | 128–191 | /16 | Medium networks (65K hosts) | 172.16.x.x |
| **C** | `110xxxxx` | 192–223 | /24 | Small networks (254 hosts) | 192.168.1.x |
| **D** | `1110xxxx` | 224–239 | — | Multicast | 224.0.0.0–239.255.255.255 |
| **E** | `1111xxxx` | 240–255 | — | Reserved/experimental | 240.0.0.0+ |

### How to Identify the Class
Look at the **first octet**: 1–126 = A, 128–191 = B, 192–223 = C, 224–239 = D, 240+ = E.

### Why It Was Wasteful
- A company needing **300 hosts** couldn't use Class C (max 254). It had to get a Class B — which has **65,534 usable hosts**. Over 65,000 addresses wasted.
- A company needing **500 hosts** also had to take a Class B. Same waste.
- No flexibility between /8, /16, and /24 — those were the only options.

### Why CIDR Replaced It
**CIDR (Classless Inter-Domain Routing)** lets you use any prefix length (/25, /26, /27, etc.), so a company needing 300 hosts gets a /23 (510 usable hosts) — minimal waste. This is the system we use today.

### Classful Addressing — Interview Q&As

**Q: What is classful addressing and why was it replaced?**

**Ideal Answer:**
"Classful addressing was IPv4's original scheme that divided IPs into fixed classes — Class A (/8), B (/16), and C (/24). It was replaced by CIDR because the fixed sizes were extremely wasteful. A company needing 300 hosts had to take a whole Class B with 65,000+ addresses. CIDR allows any prefix length, enabling efficient allocation."

**Q: How do you identify which class an IP belongs to?**

**Ideal Answer:**
"By the first octet. 1–126 is Class A, 128–191 is Class B, 192–223 is Class C, 224–239 is Class D (multicast), 240+ is Class E (reserved). For example, 172.16.5.1 starts with 172, so it's Class B."

**Q: Why is Class A/B wasteful compared to CIDR?**

**Ideal Answer:**
"Class A gives ~16 million hosts, Class B gives ~65,000, Class C gives 254. There's nothing in between. If you need 500 hosts, you must take a Class B and waste over 64,000 addresses. With CIDR, you'd use /23 (510 hosts) — exactly what you need."

---

## 3. Subnetting `MUST KNOW`

### What It Is
Dividing a large network into smaller, more manageable **sub-networks** (subnets).

### Why It Exists
- **Efficient IP allocation:** Don't waste addresses. A /24 gives 254 hosts — if you only need 30, use a /27 instead.
- **Network segmentation:** Isolate departments, reduce broadcast traffic, improve security.
- **Routing efficiency:** Smaller subnets = more specific routes.

### CIDR Notation
CIDR (Classless Inter-Domain Routing) replaces the old class-based system.

| CIDR | Subnet Mask | Network Bits | Host Bits | Total Hosts | Usable Hosts |
|------|-------------|-------------|-----------|-------------|-------------|
| /8 | 255.0.0.0 | 8 | 24 | 16,777,216 | 16,777,214 |
| /16 | 255.255.0.0 | 16 | 16 | 65,536 | 65,534 |
| /24 | 255.255.255.0 | 24 | 8 | 256 | 254 |
| /25 | 255.255.255.128 | 25 | 7 | 128 | 126 |
| /26 | 255.255.255.192 | 26 | 6 | 64 | 62 |
| /27 | 255.255.255.224 | 27 | 5 | 32 | 30 |
| /28 | 255.255.255.240 | 28 | 4 | 16 | 14 |
| /30 | 255.255.255.252 | 30 | 2 | 4 | 2 |

### Key Formulas

- **Number of hosts** = 2^(host bits) — but subtract 2 for network and broadcast addresses.
- **Usable hosts** = 2^(host bits) − 2
- **Network address** = first address (all host bits = 0)
- **Broadcast address** = last address (all host bits = 1)
- **Usable range** = network address + 1  →  broadcast address − 1

### Worked Examples

#### Example 1: `192.168.1.0/24`
```
Subnet mask:       255.255.255.0
Host bits:         32 - 24 = 8
Total addresses:   2^8 = 256
Usable hosts:      256 - 2 = 254
Network address:   192.168.1.0
Broadcast address: 192.168.1.255
Usable range:      192.168.1.1 — 192.168.1.254
```

#### Example 2: `192.168.1.0/26`
```
Subnet mask:       255.255.255.192
Host bits:         32 - 26 = 6
Total addresses:   2^6 = 64
Usable hosts:      64 - 2 = 62

This creates 4 subnets within 192.168.1.0/24:
Subnet 1: 192.168.1.0   — 192.168.1.63   (usable: .1–.62)
Subnet 2: 192.168.1.64  — 192.168.1.127  (usable: .65–.126)
Subnet 3: 192.168.1.128 — 192.168.1.191  (usable: .129–.190)
Subnet 4: 192.168.1.192 — 192.168.1.255  (usable: .193–.254)
```

#### Example 3: `10.0.0.0/28`
```
Subnet mask:       255.255.255.240
Host bits:         32 - 28 = 4
Total addresses:   2^4 = 16
Usable hosts:      16 - 2 = 14
Network address:   10.0.0.0
Broadcast address: 10.0.0.15
Usable range:      10.0.0.1 — 10.0.0.14
```

### Practice Problems

**P1:** Given `172.16.5.0/25`, find network address, broadcast address, usable range, and number of usable hosts.

**Answer:**
```
Host bits: 32 - 25 = 7 → 2^7 = 128 addresses, 126 usable
Network:   172.16.5.0
Broadcast: 172.16.5.127
Usable:    172.16.5.1 — 172.16.5.126
```

**P2:** A host has IP `192.168.10.130/26`. What subnet is it on?

**Answer:**
```
/26 = blocks of 64. 130 ÷ 64 = 2.03 → 2nd block starts at 128.
Network:   192.168.10.128
Broadcast: 192.168.10.191
The host is on subnet 192.168.10.128/26
```

**P3:** You need 50 hosts per subnet. What is the smallest subnet that works?

**Answer:**
```
Need 50 + 2 = 52 addresses minimum.
2^6 = 64 ≥ 52 → 6 host bits → /26
Answer: /26 (62 usable hosts)
```

---

## 3. Routing `MUST KNOW`

### What Routing Is
The process of **forwarding packets between different networks** based on the destination IP address.

### Router
- A **Layer 3 device** that connects different networks.
- Has a **routing table** that maps destination networks to next hops/interfaces.
- Decrements **TTL** on each packet.
- Rewrites **MAC addresses** in the frame (strips old frame, creates new frame for next hop).

### Routing Table

A router's routing table contains entries like:

| Destination Network | Next Hop | Interface | Metric |
|---|---|---|---|
| `192.168.1.0/24` | Directly connected | eth0 | 0 |
| `10.0.0.0/8` | `192.168.1.254` | eth0 | 10 |
| `0.0.0.0/0` | `203.0.113.1` | eth1 | 1 |

### Default Route
- `0.0.0.0/0` — matches **everything**.
- Used when no more specific route exists.
- "If I don't know where to send it, send it here."
- This is the gateway of last resort.

### Longest Prefix Match `MUST KNOW`

When multiple routes match a destination, the router picks the **most specific** one (longest prefix / most network bits).

**Example:** Destination = `10.1.2.5`

| Route | Match? |
|-------|--------|
| `10.0.0.0/8` | ✓ (8 bits match) |
| `10.1.0.0/16` | ✓ (16 bits match) |
| `10.1.2.0/24` | ✓ (24 bits match) ← **Winner** |
| `0.0.0.0/0` | ✓ (0 bits — default) |

The `/24` route wins because it's the most specific.

**Interview answer:** "When a router has multiple matching routes for a destination, it uses longest prefix match — the route with the most matching network bits wins. This ensures traffic is sent to the most specific, and therefore most appropriate, next hop."

### Static vs Dynamic Routing `GOOD TO KNOW`

| Feature | Static Routing | Dynamic Routing |
|---------|---------------|----------------|
| Configuration | Manually configured by admin | Automatically learned via protocols |
| Scalability | Doesn't scale — impractical for large networks | Scales well |
| Adaptability | Doesn't adapt to failures | Adapts to topology changes |
| Protocols | N/A | OSPF, BGP, RIP, EIGRP |
| Use case | Small networks, default routes | Enterprise and internet |

**Interview level:** Just know the distinction. You don't need to explain OSPF/BGP algorithms.

### TTL (Time to Live) `MUST KNOW`
- A field in the IP header, decremented by 1 at **each router hop**.
- If TTL reaches **0**, the packet is **dropped** and the router sends an **ICMP Time Exceeded** message back.
- **Why it exists:** Prevents packets from looping infinitely in the network.
- Default TTL: 64 (Linux), 128 (Windows), 255 (network devices).

### ICMP `MUST KNOW`
- **Internet Control Message Protocol** — used for error reporting and diagnostics.
- NOT used for data transfer.
- Key ICMP messages:
  - **Echo Request / Echo Reply** — used by `ping`.
  - **Destination Unreachable** — no route or port unreachable.
  - **Time Exceeded** — TTL reached 0 (used by `traceroute`).

### Ping
- Sends ICMP Echo Request → waits for ICMP Echo Reply.
- Tests: Is the destination reachable? What's the RTT?
- `ping google.com` → resolves DNS, sends ICMP echo, measures RTT.

### Traceroute
- Discovers each hop along the path to a destination.
- **How:** Sends packets with incrementing TTL (1, 2, 3...). Each router that drops the packet (TTL=0) sends back ICMP Time Exceeded, revealing its IP.
- Useful for diagnosing where packets are being dropped or delayed.

---

## 4. NAT `MUST KNOW`

### Why NAT Exists
- IPv4 has only ~4.3 billion addresses — not enough for every device.
- NAT allows multiple devices with **private IPs** to share a **single public IP** to access the internet.

### How NAT Works

```
Private Network                   NAT Router                    Internet
192.168.1.10 ──┐
192.168.1.11 ──┤── Router ──── Public IP: 203.0.113.5 ──── google.com
192.168.1.12 ──┘
```

1. Device `192.168.1.10` sends a packet to `google.com`.
2. Router receives it, replaces the **source IP** (`192.168.1.10`) with the **public IP** (`203.0.113.5`).
3. Router stores a mapping in the **NAT table**: `192.168.1.10:52431 ↔ 203.0.113.5:52431`.
4. Google responds to `203.0.113.5:52431`.
5. Router looks up NAT table, translates destination back to `192.168.1.10:52431`, forwards to the device.

### PAT (Port Address Translation) `MUST KNOW`
- Also called **NAT overload** — the most common form of NAT.
- Multiple private IPs share **one public IP**, differentiated by **port numbers**.
- Each connection gets a unique `public IP:port` mapping.
- This is what your home router does.

**Interview answer:** "NAT translates private IP addresses to a public IP address, allowing multiple internal devices to share one public IP for internet access. PAT is the most common form — it uses port numbers to distinguish between connections from different internal devices."

---

## 5. IPv6 `GOOD TO KNOW`

### Why IPv6 Exists
- IPv4: 2^32 ≈ 4.3 billion addresses — exhausted.
- IPv6: 2^128 ≈ 340 undecillion addresses — practically unlimited.

### IPv4 vs IPv6

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Address size | 32-bit | 128-bit |
| Format | Dotted decimal (`192.168.1.1`) | Hex colon-separated (`2001:0db8::1`) |
| Total addresses | ~4.3 billion | ~3.4 × 10^38 |
| NAT needed? | Yes (address shortage) | No (plenty of addresses) |
| Header | Variable length, complex | Fixed 40 bytes, simpler |
| Built-in security | Optional IPsec | Mandatory IPsec support |

### Basic IPv6 Addressing
- 128 bits, written as 8 groups of 4 hex digits: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- **Zero compression:** `::` replaces consecutive groups of zeros (only once per address).
  - `2001:0db8:0000:0000:0000:0000:0000:0001` → `2001:0db8::1`
- Loopback: `::1` (equivalent to `127.0.0.1`)

---

## Interview Questions + Answers

---

**Q1: What is a subnet mask?**

**Ideal Answer:**
"A subnet mask is a 32-bit value that separates the network portion from the host portion of an IP address. The 1-bits identify the network, the 0-bits identify the host. For example, `255.255.255.0` (/24) means the first 24 bits are the network and the last 8 bits identify hosts within that network. ANDing an IP with its subnet mask gives the network address."

---

**Q2: What is CIDR?**

**Ideal Answer:**
"CIDR — Classless Inter-Domain Routing — is a method of IP addressing that replaces the old class-based system (Class A/B/C). Instead of fixed boundaries, CIDR uses a prefix length (like /24, /26) to specify how many bits are the network portion. This allows flexible subnet sizes and efficient IP allocation. For example, /26 gives 64 addresses instead of being locked to /24's 256."

---

**Q3: Given `10.0.0.0/27`, calculate the network address, broadcast address, usable range, and number of usable hosts.**

**Ideal Answer:**
"/27 means 27 network bits, 5 host bits. 2^5 = 32 total addresses, 30 usable. Network address: `10.0.0.0`. Broadcast address: `10.0.0.31`. Usable range: `10.0.0.1` to `10.0.0.30`."

---

**Q4: What is a default gateway?**

**Ideal Answer:**
"The default gateway is the IP address of the router on your local network. When a host wants to send a packet to a destination outside its own subnet, it sends the packet to the default gateway. The gateway (router) then forwards it toward the destination. Without a default gateway, a host can only communicate within its own local subnet."

---

**Q5: What is NAT and why does it exist?**

**Ideal Answer:**
"NAT — Network Address Translation — translates private IP addresses to a public IP address. It exists because IPv4 addresses are limited (~4.3 billion). NAT allows many devices with private IPs to share a single public IP for internet access. PAT (Port Address Translation) is the most common form, using port numbers to distinguish connections from different internal devices."

---

**Q6: What is the difference between private and public IP addresses?**

**Ideal Answer:**
"Private IPs (10.x.x.x, 172.16–31.x.x, 192.168.x.x) are used within internal networks and are not routable on the internet. Public IPs are globally unique and routable on the internet. Private IPs are free to use by anyone internally; public IPs are assigned by ISPs and registries. NAT translates between them."

---

**Q7: What is longest prefix match?**

**Ideal Answer:**
"When a router has multiple routes that match a destination IP, it selects the one with the longest prefix — the most specific match. For example, if routes exist for both `10.0.0.0/8` and `10.1.0.0/16`, a packet to `10.1.2.3` matches both, but the router uses `/16` because it's more specific. This ensures the most precise forwarding decision."

---

**Q8: How does a router forward a packet?**

**Ideal Answer:**
"The router receives a frame, strips the L2 header, and reads the destination IP in the IP header. It consults its routing table and uses longest prefix match to find the best route. It determines the next hop and outgoing interface, decrements TTL, creates a new Ethernet frame with the next hop's MAC address, and sends it out. If TTL reaches 0, the packet is dropped and an ICMP Time Exceeded is sent."

> **Follow-up Q: What happens if there is no matching route?**
>
> **Ideal Answer:** "If no route matches and there's no default route (0.0.0.0/0), the router drops the packet and sends an ICMP Destination Unreachable message back to the sender."

---

**Q9: What is TTL?**

**Ideal Answer:**
"TTL — Time to Live — is a field in the IP header that's decremented by 1 at each router hop. When it reaches 0, the packet is dropped and the router sends an ICMP Time Exceeded message. TTL prevents packets from looping indefinitely in the network due to routing errors. Traceroute exploits TTL by sending packets with incrementing TTL values to discover each hop."

---

**Q10: What is ICMP? What is ping? What is traceroute?**

**Ideal Answer:**
"ICMP is the Internet Control Message Protocol, used for error reporting and diagnostics — not data transfer. Ping uses ICMP Echo Request/Reply to test if a host is reachable and measure RTT. Traceroute uses packets with incrementing TTL values — each router that drops the packet (TTL=0) sends back ICMP Time Exceeded, revealing its IP address, which maps out the path to the destination."

---

**Q11: What is the difference between static and dynamic routing?**

**Ideal Answer:**
"Static routing uses manually configured routes — simple but doesn't scale or adapt to failures. Dynamic routing uses protocols like OSPF and BGP to automatically discover and update routes — scales well and adapts to network topology changes. Static is used for small networks or default routes; dynamic is used in enterprise networks and the internet."

---

**Q12: IPv4 vs IPv6?**

**Ideal Answer:**
"IPv4 uses 32-bit addresses (~4.3 billion), IPv6 uses 128-bit addresses (practically unlimited). IPv6 was created because IPv4 addresses ran out. IPv6 has a simpler fixed-size header, doesn't need NAT (enough addresses for every device), and has mandatory IPsec support. Adoption is gradual — most of the internet still runs dual-stack (both IPv4 and IPv6)."

---

**Q13: What does it mean when a device gets a 169.254.x.x address?**

**Ideal Answer:**
"That's an APIPA (Automatic Private IP Addressing) address. It means the device tried to get an IP from a DHCP server but failed. The device auto-assigns a 169.254.x.x address so it can communicate with other devices on the same link that also have APIPA addresses, but it cannot reach the internet or other subnets."

---

### Common Mistakes

| Mistake | Correction |
|---------|------------|
| "Subnet mask is the same as an IP address" | Subnet mask defines which bits of the IP are network vs host. It's a mask, not an address. |
| "NAT changes the destination IP" | NAT (on outbound) changes the **source** IP from private to public. On return traffic, it changes the **destination** back. |
| "All host bits 0 is usable" | All host bits 0 = **network address** (not usable). All host bits 1 = **broadcast address** (not usable). |
| "Routers use MAC addresses for forwarding" | Routers use **IP addresses** and routing tables. MAC is used for local frame delivery after the routing decision. |
| "TTL is measured in seconds" | TTL is decremented at each **hop**, not by time (despite the name). |
| "IPv6 is just longer IPv4" | IPv6 also has a simplified header, no NAT requirement, built-in IPsec, and different address assignment mechanisms. |

---

### Interview Takeaways — Module 3

1. **IPv4 = 32-bit address.** Subnet mask splits it into network and host portions.
2. **Usable hosts = 2^(host bits) − 2** (subtract network address and broadcast address).
3. **CIDR** replaces classful addressing. `/24` = 256 addresses (254 usable), `/26` = 64 addresses (62 usable).
4. **Network address:** all host bits 0. **Broadcast address:** all host bits 1. **Usable range:** between them.
5. **Private IPs** (10.x, 172.16-31.x, 192.168.x) are not internet-routable. **Public IPs** are.
6. **Default gateway** = the router. Without it, you can't reach other networks.
7. **Routing = forwarding packets between networks** using routing tables and longest prefix match.
8. **Longest prefix match:** most specific route wins.
9. **TTL** prevents infinite loops — decremented at each hop, packet dropped at 0.
10. **NAT/PAT** allows many private IPs to share one public IP using port numbers.
11. **Ping = ICMP Echo Request/Reply.** Tests reachability.
12. **Traceroute** uses incrementing TTL to map each hop along the path.
13. **169.254.x.x = DHCP failed.** APIPA self-assigned address.
14. **IPv6 = 128-bit** addresses. Solves IPv4 exhaustion. No NAT needed.
