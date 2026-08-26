# Module 4.5 — TCP Performance Details

> **GOOD TO KNOW — Small but interview-relevant TCP details that fill gaps.**

---

## 1. Nagle's Algorithm `GOOD TO KNOW`

### What It Does
Nagle's algorithm **buffers small outgoing TCP segments** and coalesces them into larger ones to reduce overhead.

**Why it exists:** Without Nagle, an application sending 1-byte writes (e.g., typing in a terminal) generates a 41-byte packet for each byte (20 IP + 20 TCP + 1 data) — massive overhead. Nagle says: "If there's already unacknowledged data in flight, buffer new small writes and send them together when the ACK arrives."

### The Rule
- If there is unacknowledged data in flight **AND** the new data is smaller than MSS → **buffer it**.
- Send when: the ACK arrives, OR enough data accumulates to fill an MSS.

### Why It Can Cause Latency Issues
- Interactive applications (gaming, real-time UIs) need every small message sent **immediately**.
- With Nagle enabled, small writes are delayed waiting for ACKs — adding up to one RTT of latency.
- **Nagle + Delayed ACK** is a notorious combo: the receiver delays its ACK (hoping to piggyback it on a response), and Nagle delays the next send (waiting for that ACK). Both sides are waiting → up to 200ms+ delay.

### Solution
Disable Nagle with the `TCP_NODELAY` socket option. Common in:
- Real-time gaming
- Interactive protocols (SSH keystrokes)
- Low-latency trading systems
- Any application where latency matters more than bandwidth efficiency

**Interview answer:** "Nagle's algorithm coalesces small TCP writes into larger segments to reduce overhead. But it can cause latency issues with interactive applications because small messages are buffered. Setting TCP_NODELAY disables Nagle and sends data immediately."

---

## 2. MSS vs MTU `GOOD TO KNOW`

| | MTU | MSS |
|-|-----|-----|
| **Full Name** | Maximum Transmission Unit | Maximum Segment Size |
| **Layer** | L3 (IP) | L4 (TCP) |
| **What it measures** | Max size of entire IP packet | Max size of TCP **payload** |
| **Includes headers?** | Yes (IP header + everything above) | No (TCP data only) |
| **Ethernet default** | 1500 bytes | 1460 bytes |
| **Relationship** | — | MSS = MTU − IP header (20) − TCP header (20) |
| **Negotiated?** | No (property of the link) | Yes, during TCP handshake (SYN options) |

**Why MSS matters:** TCP negotiates MSS during the 3-way handshake. Both sides announce the largest segment they can accept. TCP then sizes its segments to fit within the smaller MSS, which ensures IP packets fit within the path MTU — avoiding fragmentation.

**Example:**
```
Ethernet MTU = 1500 bytes
IP Header = 20 bytes
TCP Header = 20 bytes
MSS = 1500 - 20 - 20 = 1460 bytes of TCP payload
```

---

## 3. TCP Multiplexing / Demultiplexing `MUST KNOW`

### Multiplexing (Sending Side)
Multiple applications on a host send data through TCP simultaneously. The transport layer **multiplexes** them by assigning different **source ports** — all data goes out through the same network interface, but each connection is distinguished by its port.

### Demultiplexing (Receiving Side)
When TCP segments arrive, the transport layer reads the **destination port** (and the full 5-tuple) to deliver each segment to the correct application/socket.

**How it works:**
- A web server listening on port 443 handles thousands of connections.
- Each incoming segment has a unique 5-tuple: (client IP, client port, server IP, 443, TCP).
- The OS uses this 5-tuple to route the segment to the correct socket (one per connection).

**Interview answer:** "TCP multiplexing lets multiple applications share the network by using different port numbers. On the receiving side, TCP demultiplexes incoming segments by looking at the destination port and the full 5-tuple to deliver data to the correct socket."

---

## 4. Piggybacking `GOOD TO KNOW`

### What It Is
**Piggybacking** means attaching an ACK to an outgoing data segment instead of sending a separate ACK-only packet.

### How It Works
- Host A sends data to Host B.
- Host B has data to send back.
- Instead of sending a separate ACK and then a separate data segment, B combines them: one segment carries both the ACK (for A's data) and B's own data.

### Why It Matters
- Reduces the number of packets on the network.
- Saves bandwidth and processing overhead.
- TCP does this automatically when possible.

### Delayed ACK
- Related concept: the receiver **delays** sending the ACK by a short time (up to 200ms) hoping to piggyback it on a data response.
- If no data is sent in that window, the ACK is sent standalone.
- **Delayed ACK + Nagle** = the bad interaction mentioned earlier.

---

## Interview Questions + Answers

---

**Q1: What is Nagle's algorithm?**

**Ideal Answer:**
"Nagle's algorithm coalesces small TCP writes into larger segments to improve network efficiency. If there's unacknowledged data in flight and the new data is small, it's buffered until the ACK arrives or a full MSS is accumulated. This reduces overhead for tiny writes but can add latency for interactive applications. Disabling it with TCP_NODELAY sends every write immediately."

> **Follow-up Q: When would you disable Nagle's algorithm?**
>
> **Ideal Answer:** "Any time low latency matters more than bandwidth efficiency — real-time gaming, SSH, interactive UIs, financial trading systems. Set TCP_NODELAY on the socket."

> **Follow-up Q: What is the Nagle + Delayed ACK problem?**
>
> **Ideal Answer:** "Nagle waits for an ACK before sending buffered data. Delayed ACK waits up to 200ms hoping to piggyback the ACK on a response. If the receiver has no data to send, both sides are waiting — causing up to 200ms of unnecessary delay. Disabling Nagle (TCP_NODELAY) breaks this deadlock."

---

**Q2: What is the difference between MSS and MTU?**

**Ideal Answer:**
"MTU is the maximum IP packet size a link can carry (1500 bytes for Ethernet) — includes all headers. MSS is the maximum TCP payload size (1460 bytes typically) — excludes IP and TCP headers. MSS = MTU minus 40 bytes (20 IP + 20 TCP). TCP negotiates MSS during the handshake so segments fit within the path MTU without IP fragmentation."

---

**Q3: How does TCP multiplexing and demultiplexing work?**

**Ideal Answer:**
"On the sending side, TCP multiplexes data from multiple applications by assigning different source ports. On the receiving side, TCP uses the 5-tuple (source IP, source port, destination IP, destination port, protocol) to demultiplex incoming segments to the correct socket. This is why one server port can handle thousands of connections — each has a unique 5-tuple."

---

**Q4: What is piggybacking in TCP?**

**Ideal Answer:**
"Piggybacking is when an ACK is combined with an outgoing data segment instead of being sent separately. If Host B has data to send back to Host A, it includes the ACK for A's data in the same segment as its own data. This reduces the number of packets and improves efficiency."

---

**Q5: What is Delayed ACK and why can it cause problems?**

**Ideal Answer:**
"Delayed ACK is when the receiver waits up to 200ms before sending an ACK, hoping to piggyback it on a data response. This reduces packet count but can interact badly with Nagle's algorithm — Nagle waits for the ACK, Delayed ACK waits for data, and both sides stall. The fix is to disable Nagle with TCP_NODELAY for latency-sensitive applications."

---

### Common Mistakes

| Mistake | Correction |
|---------|------------|
| "Nagle's algorithm makes TCP faster" | Nagle improves **bandwidth efficiency** but can increase **latency** for small writes. It's a tradeoff. |
| "MSS = MTU" | MSS = MTU − 40 bytes (IP + TCP headers). They're related but not equal. |
| "Disabling Nagle always improves performance" | It reduces latency for interactive apps but increases packet count. For bulk transfers, Nagle is beneficial. |
| "Each TCP connection needs a different server port" | Connections are distinguished by the **5-tuple**, not just the port. One port handles many connections. |

### Interview Takeaways — Module 4.5

1. **Nagle's algorithm** buffers small writes until ACK arrives or MSS is full. Disable with `TCP_NODELAY` for low latency.
2. **MSS = MTU − 40 bytes** (20 IP + 20 TCP). Negotiated during handshake. Prevents fragmentation.
3. **Multiplexing** = multiple apps share the network via different ports. **Demultiplexing** = deliver to correct socket via 5-tuple.
4. **Piggybacking** = ACK rides on a data segment. Reduces packet count.
5. **Delayed ACK + Nagle** = notorious latency interaction. Fix: `TCP_NODELAY`.
