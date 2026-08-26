# Module 3.5 — Routing Algorithms

> **GOOD TO KNOW — Brief coverage of how routing protocols actually work.**

---

## 1. Distance Vector Routing (RIP) `GOOD TO KNOW`

### Basic Idea
- Each router maintains a **routing table** with the distance (hop count) to every known destination.
- Routers **periodically share** their entire routing table with directly connected neighbors.
- Each router updates its table based on what its neighbors report — "I can reach network X in 3 hops through neighbor Y."
- Uses the **Bellman-Ford algorithm**.

### Count-to-Infinity Problem
- If a link goes down, routers keep advertising the old route through each other, incrementing the hop count each time.
- Example: Router A thinks it can reach X through B (2 hops). B thinks it can reach X through A (3 hops). They keep incrementing: 4, 5, 6... until reaching the maximum (16 in RIP = unreachable).
- **Slow convergence** — takes a long time to realize a destination is unreachable.

### Mitigations
- **Split Horizon:** Don't advertise a route back to the neighbor you learned it from. (If I learned about X from you, I won't tell you I can reach X through you.)
- **Poison Reverse:** Advertise the route back but with infinity (unreachable).
- **Route Poisoning:** Immediately advertise failed routes as unreachable.

### RIP Specifics
- Max hop count: **16** (anything ≥16 is unreachable — limits network size).
- Updates every **30 seconds**.
- Rarely used today — replaced by OSPF and BGP.

---

## 2. Link State Routing (OSPF) `GOOD TO KNOW`

### Basic Idea
- Each router discovers its **direct neighbors** and the cost of each link.
- Each router **floods** this link-state information to ALL routers in the network.
- Every router builds a **complete map** (topology) of the entire network.
- Each router independently runs **Dijkstra's shortest path algorithm** to calculate the best path to every destination.

### Why It's Better Than Distance Vector
| Aspect | Distance Vector | Link State |
|--------|----------------|------------|
| Knowledge | Only knows neighbors' tables | Complete network topology |
| Algorithm | Bellman-Ford | Dijkstra's |
| Convergence | Slow (count-to-infinity) | Fast (all routers have full picture) |
| Updates | Entire table periodically | Only changes, flooded immediately |
| Scalability | Limited (16 hops in RIP) | Better (used in enterprise networks) |

### OSPF Specifics
- Open Shortest Path First — the most common interior gateway protocol.
- No hop limit.
- Uses **areas** for scalability (Area 0 = backbone).
- Updates trigger on topology changes, not periodic.

---

## 3. BGP — One Paragraph `GOOD TO KNOW`

**How does the internet actually decide routes between ISPs?** BGP (Border Gateway Protocol) is the routing protocol used between **autonomous systems** (AS) — essentially between ISPs and large organizations. BGP is a **path vector** protocol — routers exchange full AS-path information and apply complex policies (business relationships, peering agreements) to decide which path to use. It's what glues the entire internet together. You don't need to understand BGP internals for SDE interviews — just know it exists and that it's the protocol responsible for inter-ISP routing.

---

## 4. IP Fragmentation `GOOD TO KNOW`

### When It Happens
- A packet needs to cross a link with a smaller **MTU** (Maximum Transmission Unit) than the packet size.
- Example: A 4000-byte packet hits a link with 1500-byte MTU → must be fragmented into 3 pieces.

### How It Works
- The router fragments the packet into smaller pieces, each fitting within the link's MTU.
- Each fragment has an **IP header** with the same identification number, a **fragment offset**, and a **More Fragments** flag.
- **Reassembly happens at the destination** — not at intermediate routers.

### Why It's Avoided
- Fragmentation adds overhead and complexity.
- If one fragment is lost, the entire original packet must be retransmitted.
- **Path MTU Discovery:** The sender discovers the smallest MTU along the path and sizes packets accordingly. Uses the **Don't Fragment (DF)** flag — if a router can't forward without fragmenting, it sends back ICMP "Fragmentation Needed."

### MTU vs MSS

| | MTU | MSS |
|-|-----|-----|
| Layer | L3 (IP) | L4 (TCP) |
| Includes | IP header + payload | TCP payload only |
| Ethernet default | 1500 bytes | 1460 bytes (1500 − 20 IP − 20 TCP) |
| Set by | Link layer | TCP negotiation during handshake |

**Interview point:** "MSS = MTU minus IP and TCP headers. TCP negotiates MSS during the handshake so segments fit within the MTU without fragmentation."

---

## Interview Questions + Answers

---

**Q1: What is the difference between distance vector and link state routing?**

**Ideal Answer:**
"Distance vector (RIP) — each router only knows its neighbors' routing tables and uses hop count as metric. It suffers from slow convergence and the count-to-infinity problem. Link state (OSPF) — every router has a complete topology map and runs Dijkstra's algorithm independently. It converges faster because all routers have the full picture and updates are triggered by changes, not periodic timers."

---

**Q2: What is the count-to-infinity problem?**

**Ideal Answer:**
"In distance vector routing, when a link fails, routers can keep advertising the failed route through each other with increasing hop counts, slowly counting up to the maximum (16 in RIP). This means it takes a long time to converge on the fact that the destination is unreachable. Split horizon and route poisoning mitigate this."

---

**Q3: What is split horizon?**

**Ideal Answer:**
"Split horizon is a technique to prevent routing loops in distance vector protocols. A router never advertises a route back to the neighbor it learned that route from. This breaks the loop that causes count-to-infinity."

---

**Q4: What is IP fragmentation and why is it avoided?**

**Ideal Answer:**
"Fragmentation splits a packet into smaller pieces when it's too large for a link's MTU. It's avoided because it adds overhead, complicates reassembly, and if one fragment is lost, the entire packet must be retransmitted. Modern systems use Path MTU Discovery to determine the smallest MTU along the path and size packets to avoid fragmentation entirely."

---

**Q5: What is the difference between MTU and MSS?**

**Ideal Answer:**
"MTU is the maximum transmission unit at Layer 3 — the largest IP packet a link can carry (1500 bytes for Ethernet). MSS is the maximum segment size at Layer 4 — the largest TCP payload, which equals MTU minus IP and TCP headers (typically 1460 bytes). TCP negotiates MSS during the handshake to avoid IP fragmentation."

---

**Q6: What is BGP and when is it used?**

**Ideal Answer:**
"BGP is the Border Gateway Protocol — used for routing between autonomous systems (ISPs and large organizations). It's a path vector protocol that considers full AS paths and policy-based routing decisions. It's what holds the internet together at the inter-ISP level. Interior routing (within a network) uses OSPF or similar protocols."

---

**Q7: What is Path MTU Discovery?**

**Ideal Answer:**
"Path MTU Discovery determines the smallest MTU along the entire path to a destination. The sender sets the Don't Fragment flag on packets. If a router can't forward without fragmenting, it sends back ICMP 'Fragmentation Needed' with its MTU. The sender adjusts the packet size accordingly. This avoids fragmentation entirely."

---

### Common Mistakes

| Mistake | Correction |
|---------|------------|
| "OSPF uses hop count like RIP" | OSPF uses **cost** (based on link bandwidth). RIP uses hop count. |
| "Fragmentation is reassembled at each router" | Fragments are reassembled at the **destination**, not intermediate routers. |
| "MSS and MTU are the same" | MTU includes IP+TCP headers (1500 bytes). MSS is just the TCP payload (1460 bytes). |
| "BGP is used within a company's network" | BGP is for **inter-AS** (between ISPs). Within a network, use OSPF (interior gateway protocol). |

### Interview Takeaways — Module 3.5

1. **Distance Vector (RIP):** Share tables with neighbors, Bellman-Ford, slow convergence, count-to-infinity problem.
2. **Link State (OSPF):** Flood topology, Dijkstra's, fast convergence, complete map at every router.
3. **Split Horizon** prevents routing loops by not advertising routes back to the source neighbor.
4. **BGP** = inter-ISP routing. Path vector protocol. Know it exists, skip internals.
5. **Fragmentation** splits packets at smaller MTU links. Avoided via Path MTU Discovery.
6. **MTU = 1500 (Ethernet). MSS = 1460 (MTU − IP header − TCP header).**
