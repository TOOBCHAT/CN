# Module 4 — TCP + UDP

> **MOST IMPORTANT MODULE — This is the highest-value topic for SDE interviews.**

---

## 1. TCP Fundamentals `MUST KNOW`

### What TCP Is
**Transmission Control Protocol** — a connection-oriented, reliable, ordered, byte-stream protocol at the **Transport Layer (L4)**.

**Why it exists:** The internet (IP) is unreliable — packets can be lost, duplicated, reordered, or corrupted. TCP provides reliability on top of unreliable IP.

### TCP Characteristics
- **Connection-oriented:** Must establish a connection (3-way handshake) before data transfer.
- **Reliable:** Guarantees delivery using ACKs and retransmission.
- **Ordered:** Data arrives in the order it was sent (via sequence numbers).
- **Flow control:** Prevents sender from overwhelming the receiver.
- **Congestion control:** Prevents sender from overwhelming the network.
- **Full-duplex:** Both sides can send and receive simultaneously.
- **Byte-stream:** TCP treats data as a continuous stream of bytes, not individual messages.

### TCP Segment Structure `MUST KNOW`

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |     |U|A|P|R|S|F|                                     |
| Offset| Res |R|C|S|S|Y|I|            Window Size              |
|       |     |G|K|H|T|N|N|                                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (variable)                         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                          Data                                 |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Key fields:**

| Field | Size | Purpose |
|-------|------|---------|
| **Source Port** | 16 bits | Sender's port |
| **Destination Port** | 16 bits | Receiver's port |
| **Sequence Number** | 32 bits | Position of the first byte in this segment within the data stream |
| **Acknowledgment Number** | 32 bits | Next byte the receiver expects (ACK for all bytes before this) |
| **Flags** | 6 bits | Control bits: SYN, ACK, FIN, RST, PSH, URG |
| **Window Size** | 16 bits | Receiver's available buffer space (for flow control) |
| **Checksum** | 16 bits | Error detection |

**Important flags:**
- **SYN:** Synchronize — initiates connection, carries Initial Sequence Number.
- **ACK:** Acknowledgment — confirms receipt of data.
- **FIN:** Finish — initiates connection termination.
- **RST:** Reset — abruptly terminates connection (error or rejection).
- **PSH:** Push — tell receiver to deliver data to application immediately.

---

## 2. TCP 3-Way Handshake `MUST KNOW`

> You MUST be able to explain this on a whiteboard.

### The Three Steps

```
    Client                                 Server
      |                                      |
      |  1. SYN  (seq=x)                    |
      |  ─────────────────────────────────>  |
      |                                      |
      |  2. SYN-ACK  (seq=y, ack=x+1)       |
      |  <─────────────────────────────────  |
      |                                      |
      |  3. ACK  (seq=x+1, ack=y+1)         |
      |  ─────────────────────────────────>  |
      |                                      |
      |       Connection Established         |
```

**Step-by-step:**

1. **SYN (Client → Server):**
   - Client picks a random **Initial Sequence Number (ISN)**, say `x = 1000`.
   - Sends segment with **SYN flag set**, `seq=1000`.
   - Client enters `SYN_SENT` state.

2. **SYN-ACK (Server → Client):**
   - Server picks its own ISN, say `y = 5000`.
   - Sends segment with **SYN + ACK flags set**, `seq=5000`, `ack=1001` (x+1).
   - The `ack=x+1` means "I received your SYN, I expect byte 1001 next."
   - Server enters `SYN_RECEIVED` state.

3. **ACK (Client → Server):**
   - Client sends segment with **ACK flag set**, `seq=1001`, `ack=5001` (y+1).
   - The `ack=y+1` means "I received your SYN, I expect byte 5001 next."
   - Both sides enter `ESTABLISHED` state.

### Why 3 Steps? Why Not 2?

**Interview answer:** "Both sides need to synchronize their sequence numbers. With only 2 steps, the server sends its sequence number but never gets confirmation that the client received it. The third step (ACK) confirms that the client received the server's sequence number. Without it, the server would start sending data with sequence numbers the client isn't prepared for."

### What's Exchanged During the Handshake
- **Initial Sequence Numbers** (both directions).
- **Window sizes** (for flow control).
- **MSS** (Maximum Segment Size) — largest segment each side can accept.
- **TCP options** — timestamps, window scaling, SACK, etc.

---

## 3. Reliable Transmission `MUST KNOW`

### Sequence Numbers
- Every byte in the TCP stream has a **sequence number**.
- The sequence number in a segment indicates the byte offset of the first byte in that segment.
- Example: If ISN = 1000 and first segment carries 500 bytes, `seq=1000`, next segment will have `seq=1500`.

### Acknowledgments (ACKs)
- The ACK number indicates the **next byte the receiver expects**.
- If the receiver got bytes 1000–1499, it sends `ack=1500`.
- ACKs are **cumulative**: `ack=1500` means "I have received ALL bytes up to 1499."

### Retransmission
TCP detects lost segments in two ways:

**1. Timeout (RTO — Retransmission Timeout):**
- If the sender doesn't receive an ACK within the RTO period, it retransmits.
- RTO is calculated dynamically based on measured RTT (using Karn's algorithm and exponential backoff).

**2. Duplicate ACKs (Fast Retransmit):**
- When the receiver gets an out-of-order segment, it immediately sends a **duplicate ACK** for the last in-order byte.
- After the sender receives **3 duplicate ACKs**, it retransmits the missing segment immediately — without waiting for timeout.
- This is called **fast retransmit** — much faster recovery than waiting for timeout.

### Out-of-Order Packets
- The receiver buffers out-of-order segments.
- It keeps sending duplicate ACKs for the missing segment.
- Once the missing segment arrives, the receiver delivers everything in order to the application.

```
Sender sends: [1000-1499] [1500-1999] [2000-2499] [2500-2999]
                              ↑ LOST

Receiver gets: [1000-1499] → ACK 1500
               [2000-2499] → Duplicate ACK 1500 (gap detected!)
               [2500-2999] → Duplicate ACK 1500
               (3rd dup ACK) → Duplicate ACK 1500

Sender sees 3 dup ACKs → Fast retransmit [1500-1999]

Receiver gets: [1500-1999] → ACK 3000 (everything up to 2999 received)
```

---

## 4. Flow Control `MUST KNOW`

### What It Is
Mechanism to prevent the **sender from overwhelming the receiver**. The receiver might have limited buffer space.

### How It Works
- The receiver advertises its **receive window (rwnd)** in every ACK — the amount of free buffer space.
- The sender can send at most `rwnd` bytes of unacknowledged data.
- As the receiver processes data and frees buffer, it increases `rwnd`.
- If the receiver's buffer is full, it sets `rwnd = 0` → sender stops.

### Sliding Window
- The "window" slides forward as ACKs arrive.
- Sender can transmit bytes within the window without waiting for individual ACKs.
- Window size = min(rwnd, cwnd) — the smaller of receiver window and congestion window.

```
Sent & ACKed | Sent, not ACKed | Can send | Cannot send yet
─────────────|─────────────────|──────────|──────────────────
             |<── Window ──────>|
```

### Window = 0 (Zero Window)
- Receiver's buffer is full → advertises `rwnd = 0`.
- Sender stops sending data.
- Sender periodically sends **window probe** segments to check if the window has reopened.

---

## 5. Congestion Control `MUST KNOW`

### What Congestion Is
Too many packets in the network → router queues overflow → packets dropped → retransmissions → more congestion (congestion collapse).

### Why Congestion Control Exists
Prevents the sender from overwhelming the **network**. Even if the receiver can handle the data, the network between them might not.

### Congestion Window (cwnd)
- **Sender-side** limit on unacknowledged data, separate from receiver's rwnd.
- **Actual sending rate = min(cwnd, rwnd).**

### The Four Phases

#### 1. Slow Start
- `cwnd` starts at **1 MSS** (Maximum Segment Size, typically ~1460 bytes).
- For every ACK received, `cwnd` increases by 1 MSS → effectively **doubles every RTT** (exponential growth).
- Continues until `cwnd` reaches **ssthresh** (slow start threshold).

#### 2. Congestion Avoidance
- When `cwnd ≥ ssthresh`, switch to linear growth.
- `cwnd` increases by **1 MSS per RTT** (additive increase).
- This is cautious growth to probe for available bandwidth.

#### 3. Fast Retransmit + Fast Recovery
- **3 duplicate ACKs** detected → packet loss assumed.
- **Fast retransmit:** Immediately retransmit the lost segment.
- **Fast recovery:** `ssthresh = cwnd / 2`, `cwnd = ssthresh` (halve the window, don't go back to 1).
- Continue with congestion avoidance (linear growth).

#### 4. Timeout
- **Most severe:** If RTO expires (no ACKs at all).
- `ssthresh = cwnd / 2`, **`cwnd = 1 MSS`** (back to slow start).
- Much more aggressive than fast recovery.

### Congestion Window Growth (ASCII Diagram)

```
cwnd
 │
 │                    * timeout!
 │                   *  cwnd → 1
 │               *  *
 │            * *     
 │         *  * ← congestion avoidance (linear)
 │       * *
 │     * * ← ssthresh (switch from exponential to linear)
 │   * *
 │  **  ← slow start (exponential)
 │ *
 │*
 └──────────────────────────────── Time
```

---

## 6. Reliability vs Flow Control vs Congestion Control `MUST KNOW`

> This is one of the most commonly confused distinctions in interviews.

| Mechanism | Problem It Solves | How It Works | Key Variable |
|-----------|------------------|-------------|-------------|
| **Reliability** | Packets get lost/corrupted | ACKs, sequence numbers, retransmission | Sequence/ACK numbers |
| **Flow Control** | Sender overwhelms the **receiver** | Receiver advertises window size (rwnd) | `rwnd` (receiver window) |
| **Congestion Control** | Sender overwhelms the **network** | Sender adjusts congestion window (cwnd) | `cwnd` (congestion window) |

**Interview answer:** "Reliability ensures data actually arrives — TCP uses sequence numbers, ACKs, and retransmission for this. Flow control prevents the sender from overwhelming the receiver — the receiver advertises how much buffer it has (rwnd). Congestion control prevents the sender from overwhelming the network — the sender maintains a congestion window (cwnd) that adapts based on network conditions. The actual sending rate is limited by the minimum of rwnd and cwnd."

---

## 7. TCP Connection Termination `MUST KNOW`

### 4-Way Termination

```
    Host A                                 Host B
      |                                      |
      |  1. FIN  (seq=u)                     |
      |  ─────────────────────────────────>  |  A is done sending
      |                                      |
      |  2. ACK  (ack=u+1)                  |
      |  <─────────────────────────────────  |  B acknowledges
      |                                      |
      |       (B may still send data...)     |
      |                                      |
      |  3. FIN  (seq=v)                     |
      |  <─────────────────────────────────  |  B is done sending
      |                                      |
      |  4. ACK  (ack=v+1)                  |
      |  ─────────────────────────────────>  |  A acknowledges
      |                                      |
      |  [A enters TIME_WAIT — 2×MSL]       |
```

### Why 4 Steps (Not 3)?
**TCP is full-duplex.** Each direction must be closed independently.
- After A sends FIN, A is done sending — but B might still have data to send.
- B sends ACK to acknowledge A's FIN, then continues sending any remaining data.
- When B is also done, B sends its own FIN.
- A sends the final ACK.

**Interview answer:** "TCP connections are full-duplex, so each direction closes independently. The FIN from side A says 'I'm done sending,' but side B might still have data. B ACKs the FIN, finishes sending, then sends its own FIN. That's why it takes 4 messages instead of 3."

> [!NOTE]
> Sometimes steps 2 and 3 are combined (B sends FIN+ACK together if B has no more data), making it a 3-step close. But the 4-step version is the general case.

### TIME_WAIT `MUST KNOW`

After sending the final ACK, the connection initiator enters **TIME_WAIT** for **2 × MSL** (Maximum Segment Lifetime, typically 60 seconds → TIME_WAIT = 120 seconds).

**Why TIME_WAIT exists (two reasons):**

1. **Ensure the final ACK arrives:** If the final ACK is lost, the other side will retransmit its FIN. The TIME_WAIT side must still be around to re-send the ACK.

2. **Allow old packets to expire:** Old duplicate packets from this connection might still be floating in the network. TIME_WAIT ensures these expire before a new connection using the same 5-tuple is created (which could confuse old and new packets).

---

## 8. UDP `MUST KNOW`

### What UDP Is
**User Datagram Protocol** — a connectionless, unreliable, simple protocol at the Transport Layer.

### UDP Characteristics
- **Connectionless:** No handshake. Just send.
- **Unreliable:** No ACKs, no retransmission. If a packet is lost, it's gone.
- **No ordering:** Packets may arrive in any order.
- **No flow control:** Sender can blast data as fast as it wants.
- **No congestion control:** UDP doesn't adapt to network conditions.
- **Message-oriented:** Each `send()` maps to one datagram (unlike TCP's byte-stream).

### UDP Datagram Structure

```
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Length             |           Checksum            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                            Data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Header size: 8 bytes** (vs TCP's 20+ bytes). Just source port, destination port, length, checksum.

### Why UDP Is Faster
- No connection setup (saves 1 RTT).
- No ACKs or retransmission (no overhead).
- Smaller header (8 vs 20+ bytes).
- No flow/congestion control state.

### UDP Use Cases
- **DNS:** Small query/response, speed matters, application can retry.
- **Video/audio streaming:** Late data is useless — better to skip than retransmit.
- **Online gaming:** Low latency critical. Missing one position update is fine, next one overwrites it.
- **VoIP:** Real-time voice — retransmitting a voice packet 200ms late is worse than dropping it.
- **QUIC/HTTP3:** Builds reliability on TOP of UDP (custom reliability, not TCP's).

---

## 9. TCP vs UDP `MUST KNOW`

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented (handshake) | Connectionless |
| Reliability | Reliable (ACK, retransmission) | Unreliable (best-effort) |
| Ordering | Ordered (sequence numbers) | No ordering guarantee |
| Flow Control | Yes (receiver window) | No |
| Congestion Control | Yes (cwnd, slow start) | No |
| Speed | Slower (overhead) | Faster (minimal overhead) |
| Header Size | 20–60 bytes | 8 bytes |
| Data Model | Byte stream | Message/datagram |
| Use Cases | HTTP, HTTPS, SSH, email, file transfer | DNS, streaming, gaming, VoIP, QUIC |

---

## 10. Important Comparisons `MUST KNOW`

### Flow Control vs Congestion Control

| | Flow Control | Congestion Control |
|-|---|---|
| **Problem** | Receiver overwhelmed | Network overwhelmed |
| **Who's affected** | Receiver's buffer | Routers/network links |
| **Controlled by** | Receiver (advertises rwnd) | Sender (adjusts cwnd) |
| **Mechanism** | Receiver window in TCP header | Slow start, congestion avoidance, fast recovery |

### Connection Establishment vs Connection Termination

| | Establishment | Termination |
|-|---|---|
| **Messages** | 3 (SYN, SYN-ACK, ACK) | 4 (FIN, ACK, FIN, ACK) |
| **Why different?** | Both SYN and ACK can be combined in step 2 | FIN and ACK can't always combine because B may still have data to send after ACKing A's FIN |

### Timeout vs Duplicate ACK

| | Timeout | 3 Duplicate ACKs |
|-|---|---|
| **Meaning** | No response at all — severe loss | Some packets getting through — isolated loss |
| **Response** | cwnd → 1 MSS, back to slow start | Fast retransmit + fast recovery (cwnd halved) |
| **Severity** | More severe | Less severe |

---

## Interview Questions + Answers

---

**Q1: What is TCP?**

**Ideal Answer:**
"TCP is a connection-oriented, reliable transport protocol. It establishes a connection via a 3-way handshake before data transfer, guarantees ordered and reliable delivery using sequence numbers and ACKs, and provides flow control and congestion control. It's used for HTTP, HTTPS, SSH, email — anything that needs guaranteed delivery."

---

**Q2: What is UDP?**

**Ideal Answer:**
"UDP is a connectionless, unreliable transport protocol. It has no handshake, no ACKs, no retransmission, no ordering guarantees, and no flow or congestion control. Its header is only 8 bytes, making it fast and lightweight. It's used for DNS, video streaming, gaming, and VoIP — applications where speed matters more than guaranteed delivery."

---

**Q3: TCP vs UDP?**

**Ideal Answer:**
"TCP is connection-oriented and reliable — it guarantees ordered delivery with flow and congestion control, but has more overhead. UDP is connectionless and unreliable — no guarantees, but much faster with minimal overhead. Use TCP when data must arrive correctly (HTTP, file transfer). Use UDP when speed matters and some loss is acceptable (streaming, gaming, DNS)."

---

**Q4: Explain the TCP 3-way handshake.**

**Ideal Answer:**
"The client sends a SYN with its Initial Sequence Number. The server responds with SYN-ACK, sending its own ISN and acknowledging the client's. The client sends an ACK confirming the server's ISN. After these 3 steps, both sides have synchronized sequence numbers and the connection is established. This takes 1 RTT."

> **Follow-up Q: Why not a 2-way handshake?**
>
> **Ideal Answer:** "With only 2 steps, the server sends its sequence number but the client never confirms receiving it. The server wouldn't know if the client is ready to receive data at those sequence numbers. The third ACK confirms both sides are synchronized."

> **Follow-up Q: What happens if SYN is lost?**
>
> **Ideal Answer:** "The client doesn't receive a SYN-ACK within its timeout period, so it retransmits the SYN. Typically with exponential backoff — 1s, 2s, 4s, etc. After several retries, the connection attempt fails."

> **Follow-up Q: What happens if SYN-ACK is lost?**
>
> **Ideal Answer:** "Two things happen: the client never gets the SYN-ACK, so it retransmits SYN. The server doesn't get the final ACK, so it may retransmit SYN-ACK. Eventually they synchronize, or the connection times out."

> **Follow-up Q: What if the final ACK is lost?**
>
> **Ideal Answer:** "The server remains in SYN_RECEIVED state and retransmits SYN-ACK. The client, already in ESTABLISHED state, can start sending data — and this data segment also carries an ACK, which completes the handshake for the server."

---

**Q5: What is flow control?**

**Ideal Answer:**
"Flow control prevents the sender from overwhelming the receiver. The receiver advertises its available buffer space as a window size (rwnd) in every ACK. The sender limits its unacknowledged data to rwnd bytes. If the receiver's buffer fills up, it sets rwnd to 0, and the sender pauses until the window reopens."

---

**Q6: What is congestion control?**

**Ideal Answer:**
"Congestion control prevents the sender from overwhelming the network. The sender maintains a congestion window (cwnd) that starts small (slow start, exponential growth) and grows linearly after reaching a threshold (congestion avoidance). When packet loss is detected via 3 duplicate ACKs, cwnd is halved (fast recovery). On timeout, cwnd drops to 1 MSS and restarts slow start. The actual send rate is limited by min(cwnd, rwnd)."

---

**Q7: Difference between flow control and congestion control?**

**Ideal Answer:**
"Flow control prevents overwhelming the receiver — controlled by the receiver's window size (rwnd). Congestion control prevents overwhelming the network — controlled by the sender's congestion window (cwnd). They're independent mechanisms solving different problems. The actual sending window is the minimum of both."

---

**Q8: How does TCP ensure reliable delivery?**

**Ideal Answer:**
"TCP uses sequence numbers to track every byte, ACKs to confirm receipt, and retransmission for lost data. Loss is detected two ways: timeout (RTO expires) or 3 duplicate ACKs (fast retransmit). Checksums catch corrupted segments. Sequence numbers also handle reordering — the receiver buffers out-of-order data and delivers everything in order to the application."

---

**Q9: What is fast retransmit?**

**Ideal Answer:**
"When a receiver gets out-of-order segments, it sends duplicate ACKs for the last in-order byte. After the sender receives 3 duplicate ACKs, it assumes the segment is lost and retransmits immediately — without waiting for the full timeout. This is much faster recovery because 3 dup ACKs means the network is still working (other packets are getting through), unlike a timeout which suggests severe problems."

---

**Q10: Explain TCP connection termination.**

**Ideal Answer:**
"TCP uses a 4-way close. Side A sends FIN to say it's done sending. B ACKs the FIN. B may continue sending data. When B is done, B sends its own FIN. A ACKs. It's 4 steps because TCP is full-duplex — each direction closes independently. After sending the final ACK, the initiator enters TIME_WAIT for 2×MSL."

> **Follow-up Q: Why does TIME_WAIT exist?**
>
> **Ideal Answer:** "Two reasons. First, to ensure the final ACK reaches the other side — if it's lost, the other side retransmits FIN, and the TIME_WAIT side can resend the ACK. Second, to let old duplicate packets from this connection expire before a new connection reuses the same port pair, preventing old data from being confused with new data."

---

**Q11: What happens if a TCP packet is lost during data transfer?**

**Ideal Answer:**
"The receiver notices a gap in sequence numbers, buffers any later segments, and sends duplicate ACKs for the missing segment. After 3 duplicate ACKs, the sender performs fast retransmit — resends the missing segment immediately. If no ACKs arrive at all, the sender's retransmission timer expires and it retransmits with a timeout, also reducing its congestion window aggressively."

---

**Q12: A packet arrives out of order. What happens?**

**Ideal Answer:**
"TCP buffers the out-of-order segment at the receiver and sends a duplicate ACK for the last in-order byte. When the missing segment eventually arrives, TCP delivers all the buffered data to the application in the correct order. This is why TCP guarantees ordered delivery — even though IP doesn't."

---

**Q13: Why is UDP preferred for video streaming?**

**Ideal Answer:**
"In video streaming, real-time playback matters more than perfect delivery. A retransmitted video frame that arrives 200ms late is useless — the playback has already moved on. UDP's lack of retransmission and connection overhead means lower latency. The application can handle minor losses (skip the frame, reduce quality) rather than stalling for retransmission."

---

**Q14: Can you build reliability on top of UDP?**

**Ideal Answer:**
"Yes — QUIC does exactly this. QUIC is a transport protocol built over UDP that provides reliability, ordering, encryption, and multiplexing. It's used by HTTP/3. The advantage over TCP is that QUIC implements these features in user space (not the kernel), can be updated faster, and handles multiplexing better — individual stream loss doesn't block other streams."

---

**Q15: Explain the congestion window behavior from connection start to a congestion event.**

**Ideal Answer:**
"cwnd starts at 1 MSS. During slow start, it doubles every RTT (exponential growth) until reaching ssthresh. Then congestion avoidance takes over — cwnd grows by 1 MSS per RTT (linear). When 3 duplicate ACKs occur: ssthresh = cwnd/2, cwnd = ssthresh, continue linear growth (fast recovery). When a timeout occurs: ssthresh = cwnd/2, cwnd = 1 MSS, restart slow start."

---

**Q16: What is the sliding window?**

**Ideal Answer:**
"The sliding window is TCP's mechanism for efficient data transfer. The sender can transmit multiple segments without waiting for individual ACKs — up to the window size. As ACKs arrive, the window slides forward, allowing new data to be sent. The window size is min(rwnd, cwnd) — the smaller of the receiver's buffer and the sender's congestion estimate."

---

### Common Mistakes

| Mistake | Correction |
|---------|------------|
| "TCP is faster than UDP" | TCP is **slower** — it has handshake, ACK, retransmission overhead. UDP is faster because it has none of that. |
| "Flow control and congestion control are the same" | Flow control = receiver limit (rwnd). Congestion control = network limit (cwnd). Different problems, different mechanisms. |
| "TCP guarantees data won't be lost" | TCP guarantees **delivery** through retransmission, but packets can still be lost in the network. TCP detects and recovers from loss. |
| "The 3-way handshake takes 3 RTTs" | It takes **1 RTT** (SYN → SYN-ACK → ACK). The SYN and final ACK are in the same RTT. Actually: SYN takes half RTT to reach server, SYN-ACK takes half RTT back = 1 RTT, then ACK goes with the first data. |
| "UDP has no checksum" | UDP **does** have a checksum for error detection (optional in IPv4, mandatory in IPv6). It just doesn't retransmit if the check fails. |
| "3 duplicate ACKs = 3 ACKs total" | 3 **duplicate** ACKs means 3 ACKs with the **same** acknowledgment number (plus the original ACK = 4 total ACKs for the same byte). |
| "TIME_WAIT is a bug / wastes resources" | TIME_WAIT is essential for correctness. Without it, old packets could corrupt new connections. |
| "TCP connection termination always takes 4 messages" | It can take 3 if the receiver piggybacks FIN+ACK together when it has no more data to send. |

---

### Interview Takeaways — Module 4

1. **TCP = reliable, ordered, connection-oriented. UDP = fast, unreliable, connectionless.**
2. **3-way handshake: SYN → SYN-ACK → ACK.** Synchronizes sequence numbers. Takes 1 RTT.
3. **Why 3 steps?** Both sides must confirm each other's sequence numbers.
4. **Sequence numbers** track byte position. **ACKs** confirm receipt and indicate next expected byte.
5. **Loss detection:** Timeout (severe — cwnd → 1) or 3 duplicate ACKs (fast retransmit — cwnd halved).
6. **Flow control = receiver limit (rwnd). Congestion control = network limit (cwnd). Different problems.**
7. **Actual send window = min(rwnd, cwnd).**
8. **Slow start:** Exponential growth until ssthresh. **Congestion avoidance:** Linear growth after ssthresh.
9. **4-way termination: FIN, ACK, FIN, ACK.** Each direction closes independently (full-duplex).
10. **TIME_WAIT (2×MSL):** Ensures final ACK arrives + lets old packets expire.
11. **UDP header = 8 bytes. TCP header = 20+ bytes.**
12. **UDP use cases:** DNS, streaming, gaming, VoIP, QUIC.
13. **TCP treats data as a byte stream. UDP preserves message boundaries.**
14. **Fast retransmit** after 3 dup ACKs is faster than timeout-based recovery.
15. **QUIC (HTTP/3)** builds TCP-like reliability over UDP with better multiplexing.
