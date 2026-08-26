# Module 2.5 — Medium Access & ARQ

> **GOOD TO KNOW — Closes gaps from the Data Link Layer module. Keep answers brief.**

---

## 1. Medium Access Control `GOOD TO KNOW`

### CSMA/CD (Collision Detection) — Wired Ethernet
- **What:** Before sending, a device **listens** to the medium. If idle, transmit. If two devices transmit simultaneously → **collision detected**.
- **How:** While transmitting, the sender monitors for collision. If detected, both stop, send a **jam signal**, wait a **random backoff time**, then retry.
- **Why it matters:** This is how old (half-duplex) Ethernet worked. Modern full-duplex switched Ethernet doesn't need CSMA/CD — switches eliminate collisions.
- **Interview point:** "CSMA/CD was used in shared Ethernet. Modern switches give each port a dedicated link, so collisions don't happen. CSMA/CD is effectively obsolete in wired networks."

### CSMA/CA (Collision Avoidance) — Wi-Fi (802.11)
- **What:** Wi-Fi **can't detect** collisions (radio signals interfere), so it **avoids** them instead.
- **How:** Before sending, listen. If idle, wait a random backoff. Optionally use **RTS/CTS** (Request to Send / Clear to Send) to reserve the channel. Send data. Wait for **ACK** from receiver.
- **Why Wi-Fi can't use CD:** A wireless sender can't listen while transmitting — its own signal drowns out everything else (the "hidden terminal" problem).
- **Interview point:** "Wi-Fi uses CSMA/CA because wireless devices can't detect collisions during transmission. It avoids collisions using random backoff and optional RTS/CTS handshake."

---

## 2. ARQ Protocols (Automatic Repeat reQuest) `GOOD TO KNOW`

ARQ protocols define **how a sender retransmits lost or corrupted frames**. These are the building blocks behind TCP's reliability.

### Stop-and-Wait
- Send **one frame**, wait for **ACK**, then send next.
- Simple but extremely slow — link is idle while waiting for ACK.
- Utilization is terrible on high-latency links.

### Go-Back-N
- Sender can send up to **N frames** without waiting (sliding window of size N).
- If a frame is lost, the receiver **discards all subsequent frames** (even if they arrived correctly).
- Sender retransmits from the lost frame onward — "goes back N."
- Simpler receiver (no buffering), but wastes bandwidth re-sending correct frames.

### Selective Repeat
- Sender can send up to **N frames** without waiting.
- Receiver **buffers out-of-order frames** and only requests retransmission of the specific lost frame.
- More efficient than Go-Back-N — only the lost frame is retransmitted.
- More complex receiver (needs buffer for out-of-order frames).

### How They Relate to TCP
TCP's actual behavior is closest to **Selective Repeat** — it buffers out-of-order segments, sends duplicate ACKs for missing segments, and the sender retransmits only the specific lost segment (fast retransmit). TCP's sliding window is the practical evolution of these ARQ concepts.

| Protocol | Window Size | On Loss | Efficiency | TCP Equivalent |
|----------|-----------|---------|------------|----------------|
| Stop-and-Wait | 1 | Retransmit that one frame | Very low | N/A |
| Go-Back-N | N | Retransmit from lost frame onward | Medium | N/A |
| Selective Repeat | N | Retransmit only lost frame | High | TCP's actual behavior |

---

## 3. Error Detection `GOOD TO KNOW`

### Checksum (IP / TCP / UDP)
- A simple sum-based computation over the header (and sometimes data).
- **Detects:** Accidental bit errors during transmission.
- **Doesn't detect:** Deliberate tampering (not cryptographic), some multi-bit errors.
- IP, TCP, and UDP all have checksum fields. If the checksum fails, the segment/packet is dropped.

### CRC (Cyclic Redundancy Check)
- Used in Ethernet frames (the FCS field).
- **Stronger** error detection than a simple checksum — catches more error patterns.
- Practical understanding: CRC is a polynomial-based check appended to the frame. Receiver recomputes it — if it doesn't match, the frame is silently dropped.
- **Skip the math** — just know CRC is better than checksum and is used at Layer 2.

---

## 4. Network Device Comparison `MUST KNOW`

| Device | Layer | Addresses | Function | Domain Impact |
|--------|-------|-----------|----------|---------------|
| **Repeater** | L1 | None | Amplifies/regenerates signal | Extends collision domain |
| **Hub** | L1 | None | Repeater with multiple ports — floods to all | One collision domain, one broadcast domain |
| **Bridge** | L2 | MAC | Connects 2 LAN segments, filters by MAC | Separates collision domains |
| **Switch** | L2 | MAC | Multi-port bridge — learns MACs, forwards selectively | 1 collision domain per port, 1 broadcast domain |
| **Router** | L3 | IP | Forwards packets between networks using routing table | Separates broadcast domains |
| **Gateway** | L7+ | All | Protocol converter (e.g., translates between different network architectures) | Application-level translation |

**Interview shortcut:** "Repeater/hub = L1, dumb signal. Bridge/switch = L2, MAC-based forwarding. Router = L3, IP-based routing. Gateway = protocol translation across different systems."

> [!NOTE]
> In practice, "gateway" often just means "default gateway" (your router). The formal L7 gateway definition is rarely asked — just be aware it exists.

---

## Interview Questions + Answers

---

**Q1: What is the difference between CSMA/CD and CSMA/CA?**

**Ideal Answer:**
"CSMA/CD (Collision Detection) is used in wired Ethernet — devices detect collisions during transmission and retry after a random backoff. CSMA/CA (Collision Avoidance) is used in Wi-Fi — wireless devices can't detect collisions (their own signal drowns out others), so they avoid collisions using random backoff and optional RTS/CTS. Modern switched Ethernet doesn't need CSMA/CD since switches eliminate collisions."

---

**Q2: What are the ARQ protocols and how do they differ?**

**Ideal Answer:**
"Stop-and-Wait sends one frame at a time and waits for ACK — simple but slow. Go-Back-N uses a sliding window of N frames but retransmits everything from the lost frame onward — wastes bandwidth. Selective Repeat also uses a window of N but retransmits only the specific lost frame — most efficient. TCP's behavior is closest to Selective Repeat."

---

**Q3: What is the difference between a checksum and CRC?**

**Ideal Answer:**
"Both detect transmission errors, but CRC is much stronger. Checksum is a simple sum used in IP/TCP/UDP headers — it catches basic bit errors. CRC is a polynomial-based check used in Ethernet frames — it detects a wider range of error patterns. Neither is cryptographic — they detect accidental errors, not deliberate tampering."

---

**Q4: How does TCP's reliability relate to these ARQ protocols?**

**Ideal Answer:**
"TCP's sliding window and retransmission are a practical evolution of ARQ protocols. TCP behaves like Selective Repeat — it buffers out-of-order segments, sends duplicate ACKs for gaps, and the sender retransmits only the specific lost segment via fast retransmit, rather than resending everything after the lost segment."

---

**Q5: Repeater vs hub vs bridge vs switch vs router?**

**Ideal Answer:**
"Repeater (L1) amplifies signals. Hub (L1) is a multi-port repeater — floods to all ports. Bridge (L2) connects two segments and filters by MAC. Switch (L2) is a multi-port bridge — learns MACs and forwards selectively. Router (L3) forwards packets between networks using IP and routing tables. Each higher-layer device provides more intelligence and isolation."

---

**Q6: Why doesn't modern Ethernet need CSMA/CD?**

**Ideal Answer:**
"Modern Ethernet uses full-duplex switched connections. Each device has a dedicated link to the switch port — there's no shared medium to collide on. Collisions only happen on shared half-duplex links, which are obsolete. CSMA/CD is still in the Ethernet spec but effectively unused."

---

### Common Mistakes

| Mistake | Correction |
|---------|------------|
| "Wi-Fi uses CSMA/CD" | Wi-Fi uses **CSMA/CA** — it can't detect collisions during transmission. |
| "Go-Back-N is what TCP uses" | TCP uses **Selective Repeat** behavior — retransmits only the lost segment, not everything after it. |
| "CRC corrects errors" | CRC **detects** errors. It doesn't correct them — the frame is dropped. |
| "A bridge and a switch are different things" | A switch is essentially a **multi-port bridge**. Same concept, more ports. |

### Interview Takeaways — Module 2.5

1. **CSMA/CD** = wired Ethernet (obsolete with full-duplex switches). **CSMA/CA** = Wi-Fi (can't detect, so avoids).
2. **Stop-and-Wait** (1 at a time) → **Go-Back-N** (window, retransmit all) → **Selective Repeat** (window, retransmit only lost). TCP ≈ Selective Repeat.
3. **Checksum** = simple error detection (IP/TCP/UDP). **CRC** = stronger error detection (Ethernet frames).
4. **Device hierarchy:** Repeater/Hub (L1) → Bridge/Switch (L2) → Router (L3) → Gateway (L7).
5. Switches eliminated collisions → CSMA/CD is obsolete in modern wired networks.
