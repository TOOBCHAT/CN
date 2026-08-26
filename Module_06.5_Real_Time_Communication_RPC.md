# Module 6.5 — Real-Time Communication + RPC

> **MUST KNOW for full-stack roles — covers WebSockets, SSE, long polling, and RPC/gRPC.**

---

## 1. The Problem: Real-Time Updates Over HTTP `MUST KNOW`

Standard HTTP is **request-response** — the client asks, the server answers. But many applications need the server to **push** data to the client without being asked: chat messages, live scores, stock prices, notifications.

HTTP was never designed for this. These patterns solve that gap.

---

## 2. Long Polling `MUST KNOW`

### How It Works
1. Client sends an HTTP request.
2. Server **holds the request open** (doesn't respond immediately).
3. When new data is available, the server sends the response.
4. Client immediately sends another request.
5. Repeat.

```
Client                          Server
  |── GET /updates ──────────>|
  |        (server waits...)   |
  |        (new data arrives)  |
  |<── 200 OK {data} ─────────|
  |── GET /updates ──────────>|  ← immediately re-request
  |        (server waits...)   |
  ...
```

### Pros & Cons
- ✅ Works with any HTTP infrastructure (proxies, load balancers).
- ✅ Simple to implement.
- ❌ Each "push" requires a new HTTP request (overhead).
- ❌ Connection held open consumes server resources.
- ❌ Not truly real-time — there's latency between responses.

---

## 3. Server-Sent Events (SSE) `MUST KNOW`

### How It Works
1. Client opens a **single long-lived HTTP connection** (`EventSource` API).
2. Server sends events **one-way** (server → client) over this connection.
3. Data streams as `text/event-stream` content type.
4. Connection stays open — server pushes whenever it wants.

```
Client                          Server
  |── GET /events ──────────>|
  |   Accept: text/event-stream
  |                            |
  |<── data: {"msg": "hi"} ───|
  |<── data: {"msg": "hello"}─|
  |<── data: {"msg": "bye"} ──|
  ...    (server keeps pushing)
```

### Key Features
- **One-way:** Server → client only. Client sends data via separate HTTP requests.
- **Auto-reconnect:** The browser automatically reconnects if the connection drops.
- **Simple:** Uses standard HTTP — works with proxies, firewalls, load balancers.
- **Text-only:** Data is always text (UTF-8).

### Pros & Cons
- ✅ Simple, built on HTTP, auto-reconnect.
- ✅ Great for server-push use cases (notifications, live feeds).
- ❌ **One-way only** — can't send from client over the same connection.
- ❌ Text-only (no binary).
- ❌ Limited to ~6 connections per domain in HTTP/1.1 (not an issue with HTTP/2).

---

## 4. WebSockets `MUST KNOW`

### What It Is
A **full-duplex, bidirectional** communication protocol over a single persistent TCP connection. Both client and server can send messages at any time.

### How It Works

#### The Handshake (HTTP Upgrade)
WebSocket starts as a regular HTTP request, then **upgrades** the connection:

```
Client                                      Server
  |── GET /chat HTTP/1.1 ─────────────────>|
  |   Upgrade: websocket                    |
  |   Connection: Upgrade                   |
  |   Sec-WebSocket-Key: dGhlIHNhbXBsZS4=  |
  |                                          |
  |<── HTTP/1.1 101 Switching Protocols ────|
  |    Upgrade: websocket                    |
  |    Connection: Upgrade                   |
  |    Sec-WebSocket-Accept: s3pPLM...       |
  |                                          |
  |═══ Full-duplex WebSocket connection ════|
  |                                          |
  |── {"type": "message", "text": "hi"} ──>|
  |<── {"type": "message", "text": "hey"} ──|
  |── {"type": "typing"} ─────────────────>|
  |<── {"type": "message", "text": "sup"} ──|
```

1. Client sends HTTP GET with `Upgrade: websocket` header.
2. Server responds with **101 Switching Protocols**.
3. The TCP connection is now a WebSocket — no more HTTP framing.
4. Both sides can send messages freely (binary or text).

### Key Features
- **Full-duplex:** Both sides send/receive simultaneously.
- **Low overhead:** After the handshake, messages have tiny frames (2-14 bytes header) vs HTTP's repeated headers.
- **Binary and text:** Supports both.
- **Persistent:** One connection for the entire session.

### When to Use WebSockets
- **Chat applications** (Slack, Discord)
- **Real-time collaboration** (Google Docs)
- **Live gaming**
- **Live dashboards** (stock tickers, monitoring)
- Any scenario needing **frequent bidirectional** communication.

### Pros & Cons
- ✅ True bidirectional, low latency, low overhead.
- ✅ Supports binary and text.
- ❌ More complex than SSE.
- ❌ Not all proxies/firewalls handle WebSockets well.
- ❌ No built-in reconnection (must implement in application).
- ❌ Stateful connection — harder to load balance.

---

## 5. WebSocket vs SSE vs Long Polling `MUST KNOW`

| Feature | Long Polling | SSE | WebSocket |
|---------|-------------|-----|-----------|
| **Direction** | Server → Client (simulated) | Server → Client | **Bidirectional** |
| **Protocol** | HTTP | HTTP | WebSocket (starts as HTTP) |
| **Connection** | New request per event | Single persistent HTTP | Single persistent TCP |
| **Data format** | Any | Text only | Text + Binary |
| **Auto-reconnect** | Manual | Built-in | Manual |
| **Overhead** | High (HTTP headers each time) | Low (streaming) | Very low (tiny frames) |
| **Complexity** | Low | Low | Medium |
| **Best for** | Simple, infrequent updates | Server-push (notifications, feeds) | Bidirectional real-time (chat, gaming) |

### Decision Framework
- **Need server → client only?** Use **SSE** (simpler, auto-reconnect, built on HTTP).
- **Need bidirectional?** Use **WebSocket** (chat, gaming, collaboration).
- **Can't use SSE or WebSocket?** Fall back to **long polling** (works everywhere).

---

## 6. REST vs RPC vs gRPC `MUST KNOW`

### REST (Representational State Transfer)
- **Resource-oriented:** URLs represent resources (`/users/123`), HTTP methods represent actions (GET, POST, PUT, DELETE).
- **Stateless:** Each request contains all information needed.
- **Text-based:** Typically JSON over HTTP.
- **Standard HTTP:** Works with caches, proxies, browsers.
- **Best for:** Public APIs, CRUD-heavy services, web frontends.

### RPC (Remote Procedure Call)
- **Action-oriented:** Client calls a function on the server as if it were local.
- **Concept:** Client stub serializes (marshals) the function call and arguments → sends over network → server stub deserializes (unmarshals) → executes → returns result.
- **Various implementations:** XML-RPC, JSON-RPC, gRPC.

### gRPC (Google RPC)
- Modern RPC framework by Google.
- Uses **Protocol Buffers (protobuf)** for serialization — binary, compact, fast.
- Runs over **HTTP/2** — multiplexing, header compression, streaming.
- Supports **4 communication patterns:**
  - Unary (request-response)
  - Server streaming
  - Client streaming
  - Bidirectional streaming
- **Code generation:** Define service in `.proto` file → generate client/server stubs automatically.

### Comparison

| Feature | REST | RPC / gRPC |
|---------|------|------------|
| **Style** | Resource-oriented (nouns) | Action-oriented (verbs) |
| **Data format** | JSON (text) | Protobuf (binary) — gRPC |
| **Transport** | HTTP/1.1 or HTTP/2 | HTTP/2 (gRPC) |
| **Performance** | Slower (text parsing, HTTP overhead) | Faster (binary, multiplexing) |
| **Streaming** | Not native | Built-in (gRPC) |
| **Browser support** | Native | Limited (gRPC needs gRPC-Web proxy) |
| **Best for** | Public APIs, web clients | Internal microservices, low-latency |
| **Tooling** | curl, browser, any HTTP client | Generated stubs, specific clients |

### When to Choose

- **REST:** Public APIs, browser-facing, CRUD, simplicity.
- **gRPC:** Internal microservice communication, high performance, streaming, polyglot services.
- **WebSocket:** When you need persistent bidirectional real-time communication (chat, live updates).

---

## 7. Basic RPC Concept `GOOD TO KNOW`

```
Client App                                    Server App
     |                                            |
     |── call getUserById(123) ──>|               |
     |                             |               |
     |     [Client Stub]          |               |
     |     Marshal: serialize     |               |
     |     function name + args   |               |
     |     into bytes             |               |
     |                             |               |
     |── network request ─────────────────────────>|
     |                             |               |
     |                             |    [Server Stub]
     |                             |    Unmarshal: deserialize
     |                             |    Call actual getUserById(123)
     |                             |    Get result
     |                             |    Marshal result
     |                             |               |
     |<── network response ────────────────────────|
     |                             |               |
     |     [Client Stub]          |               |
     |     Unmarshal result       |               |
     |     Return User object     |               |
     |                             |               |
     |<── User{id:123, name:"A"} ──|              |
```

**Interview answer:** "RPC lets a client call a function on a remote server as if it were a local call. The client stub serializes the function name and arguments, sends them over the network, the server stub deserializes them, executes the function, serializes the result, and sends it back. gRPC uses Protocol Buffers for efficient binary serialization and HTTP/2 for transport."

---

## Interview Questions + Answers

---

**Q1: What is the difference between WebSocket, SSE, and long polling?**

**Ideal Answer:**
"Long polling simulates server push by having the client re-request after each response — it's simple but has high overhead. SSE provides one-way server-to-client streaming over a single HTTP connection with auto-reconnect — great for notifications and feeds. WebSocket provides full-duplex bidirectional communication over a single TCP connection with minimal overhead — best for chat, gaming, and real-time collaboration. Use SSE when you only need server push; use WebSocket when you need bidirectional."

---

**Q2: How does a WebSocket connection start?**

**Ideal Answer:**
"It starts as a regular HTTP GET request with an `Upgrade: websocket` header. The server responds with 101 Switching Protocols, and the connection upgrades from HTTP to WebSocket. After that, the connection is a persistent, full-duplex TCP connection where both sides can send messages with minimal framing overhead."

---

**Q3: When would you use WebSockets over regular HTTP?**

**Ideal Answer:**
"When you need frequent, bidirectional, low-latency communication. Chat applications, real-time collaboration (like Google Docs), live gaming, stock tickers, and monitoring dashboards. If you only need occasional server pushes, SSE is simpler. If you only need request-response, regular HTTP is sufficient."

---

**Q4: What is the difference between REST and gRPC?**

**Ideal Answer:**
"REST is resource-oriented — URLs represent resources, uses JSON over HTTP, works with browsers and caches. gRPC is action-oriented — calls remote functions, uses binary Protocol Buffers over HTTP/2, supports streaming. REST is better for public APIs and browser clients. gRPC is better for internal microservice communication where performance and streaming matter."

---

**Q5: What is RPC?**

**Ideal Answer:**
"Remote Procedure Call — the client calls a function on a remote server as if it were local. The client stub serializes the function name and arguments (marshalling), sends them over the network, the server stub deserializes and executes the function, then sends the result back. gRPC is Google's modern implementation using Protocol Buffers and HTTP/2."

---

**Q6: When would you choose SSE over WebSocket?**

**Ideal Answer:**
"When you only need server-to-client push and don't need the client to send data over the same connection. SSE is simpler — it uses standard HTTP, has built-in reconnection, and works easily with proxies and load balancers. Notification feeds, live scoreboards, and log streaming are good SSE use cases."

---

**Q7: Why does gRPC use HTTP/2?**

**Ideal Answer:**
"HTTP/2 provides multiplexing (multiple concurrent RPCs over one connection), header compression (reduced overhead), and native streaming support (server, client, and bidirectional). These features make HTTP/2 ideal for high-performance inter-service communication. It also means gRPC can leverage existing HTTP/2 infrastructure."

---

**Q8: How is WebSocket different from HTTP?**

**Ideal Answer:**
"HTTP is request-response — client asks, server answers, one at a time. WebSocket is full-duplex — both sides can send messages at any time over a persistent connection. HTTP adds significant header overhead per request. WebSocket frames have tiny headers (2-14 bytes). WebSocket starts with an HTTP upgrade handshake but then drops HTTP entirely."

---

**Q9: Can WebSocket work through a load balancer?**

**Ideal Answer:**
"Yes, but it requires the load balancer to support WebSocket connections — L7 load balancers need to handle the HTTP upgrade and maintain the persistent connection. Since WebSocket connections are stateful (tied to a specific backend server), you typically use sticky sessions (IP hash or session affinity) to ensure all messages go to the same server."

---

**Q10: What is Protocol Buffers?**

**Ideal Answer:**
"Protocol Buffers (protobuf) is Google's binary serialization format. You define data structures in `.proto` files, and protobuf generates code to serialize/deserialize in multiple languages. It's much faster and smaller than JSON — binary encoding vs text, and fields are identified by number not name. gRPC uses protobuf as its default serialization."

---

### Common Mistakes

| Mistake | Correction |
|---------|------------|
| "WebSocket replaces HTTP" | WebSocket **complements** HTTP — use it when you need bidirectional real-time. Most web communication is still HTTP. |
| "SSE is bidirectional" | SSE is **server-to-client only**. For bidirectional, use WebSocket. |
| "Long polling and WebSocket are the same" | Long polling uses repeated HTTP requests. WebSocket is a persistent full-duplex connection. Very different overhead and latency. |
| "gRPC works in browsers natively" | gRPC needs a **gRPC-Web proxy** for browser clients. Native browser support is for REST/HTTP. |
| "REST is always better for APIs" | REST is better for public/external APIs. For internal microservices, gRPC offers better performance and streaming. |

### Interview Takeaways — Module 6.5

1. **Long polling:** Server holds request, responds when data ready, client re-requests. Simple but high overhead.
2. **SSE:** One-way server push over persistent HTTP. Auto-reconnect. Great for feeds/notifications.
3. **WebSocket:** Full-duplex bidirectional over persistent TCP. Starts with HTTP upgrade (101). Best for chat/gaming/collab.
4. **Decision:** Server-push only → SSE. Bidirectional → WebSocket. Fallback → Long polling.
5. **REST:** Resource-oriented, JSON, HTTP, public APIs. **gRPC:** Action-oriented, protobuf, HTTP/2, internal services.
6. **RPC concept:** Client stub marshals → network → server stub unmarshals → execute → return.
7. **WebSocket handshake:** HTTP GET with `Upgrade: websocket` → 101 Switching Protocols → full-duplex.
8. **gRPC advantages:** Binary (fast), HTTP/2 (multiplexing), streaming, code generation.
