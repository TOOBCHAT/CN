# Module 8 — Sockets + Ports

> **Relatively short module — focused on interview essentials.**

---

## 1. Port Numbers `MUST KNOW`

A **16-bit number** (0–65535) that identifies a specific process or service on a machine.

### Port Ranges

| Range | Name | Purpose | Examples |
|-------|------|---------|---------|
| 0–1023 | **Well-known ports** | Reserved for standard services | HTTP=80, HTTPS=443, DNS=53, SSH=22, FTP=21, SMTP=25 |
| 1024–49151 | **Registered ports** | Application-specific (IANA-registered) | MySQL=3306, PostgreSQL=5432, Redis=6379 |
| 49152–65535 | **Dynamic/Ephemeral ports** | OS assigns these to client connections | Your browser uses a random port from this range |

### Important Well-Known Ports

| Port | Protocol | Service |
|------|----------|---------|
| 20, 21 | TCP | FTP (data, control) |
| 22 | TCP | SSH |
| 25 | TCP | SMTP (email sending) |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |
| 3306 | TCP | MySQL |
| 5432 | TCP | PostgreSQL |
| 6379 | TCP | Redis |
| 8080 | TCP | HTTP alternate (dev servers) |

**Key point:** TCP and UDP port spaces are **independent**. TCP port 53 and UDP port 53 are different — DNS uses both (UDP for queries, TCP for zone transfers).

---

## 2. Sockets `MUST KNOW`

### What a Socket Is
A **socket** is an endpoint for network communication, defined by:

```
Socket = (IP Address, Port Number)
```

A socket is the interface between the application layer and the transport layer. It's how your code talks to the network.

### Connection Identity
A complete TCP connection is uniquely identified by a **5-tuple**:

```
(Source IP, Source Port, Destination IP, Destination Port, Protocol)
```

This is why a single server port (e.g., 443) can handle **thousands of concurrent connections** — each connection has a unique combination of source IP + source port.

### Client Socket vs Server Socket

| | Client | Server |
|-|--------|--------|
| Port | **Ephemeral** (random, assigned by OS, e.g., 52431) | **Well-known** (fixed, e.g., 80, 443) |
| Role | Initiates connection | Listens for incoming connections |
| Quantity | One socket per connection | One listening socket + one new socket per accepted connection |

---

## 3. Socket Lifecycle `MUST KNOW`

### Server Side

```python
# 1. Create socket
server_socket = socket(AF_INET, SOCK_STREAM)   # TCP socket

# 2. Bind to address
server_socket.bind(("0.0.0.0", 8080))          # Listen on all interfaces, port 8080

# 3. Listen for connections
server_socket.listen(backlog=128)               # Queue up to 128 pending connections

# 4. Accept a connection (blocks until a client connects)
client_socket, client_addr = server_socket.accept()  # Returns NEW socket for this client

# 5. Exchange data
data = client_socket.recv(4096)                 # Read from client
client_socket.send(response)                    # Send to client

# 6. Close
client_socket.close()                           # Close this client's connection
```

### Client Side

```python
# 1. Create socket
client_socket = socket(AF_INET, SOCK_STREAM)   # TCP socket

# 2. Connect to server (triggers TCP 3-way handshake)
client_socket.connect(("example.com", 8080))    # OS assigns ephemeral source port

# 3. Exchange data
client_socket.send(request)                     # Send to server
data = client_socket.recv(4096)                 # Read from server

# 4. Close
client_socket.close()                           # Triggers TCP 4-way termination
```

### Key Functions

| Function | What It Does | Side |
|----------|-------------|------|
| `socket()` | Creates a new socket | Both |
| `bind(IP, port)` | Attaches socket to a specific address and port | Server |
| `listen()` | Marks socket as passive — ready to accept connections | Server |
| `accept()` | Accepts incoming connection, returns a **new** socket for that client | Server |
| `connect()` | Initiates TCP handshake to server | Client |
| `send()`/`recv()` | Send and receive data | Both |
| `close()` | Closes connection (triggers TCP FIN) | Both |

> [!IMPORTANT]
> **`accept()` returns a NEW socket.** The original listening socket continues listening for more connections. Each client gets its own dedicated socket. This is how a server handles multiple clients on the same port.

---

## 4. How Sockets Relate to TCP, HTTP, and Backend Servers `MUST KNOW`

### Sockets ↔ TCP
- `connect()` triggers the **TCP 3-way handshake**.
- `send()`/`recv()` transfer data through TCP's reliable byte stream.
- `close()` triggers the **TCP 4-way termination**.
- TCP handles all the reliability, flow control, and congestion control transparently.

### Sockets ↔ HTTP
- When your browser makes an HTTP request, under the hood:
  1. Browser creates a TCP socket.
  2. `connect()` to server IP:443 (TCP handshake).
  3. TLS handshake over the socket.
  4. `send()` the HTTP request bytes (encrypted).
  5. `recv()` the HTTP response bytes.
  6. Parse the response.
- HTTP libraries abstract this away, but sockets are what actually happen.

### Sockets ↔ Backend Servers
- A web server (Nginx, Express, Flask) calls `bind()` on port 80/443, then `listen()`.
- Each incoming request → `accept()` → new socket → read HTTP request → process → send HTTP response → close.
- **Multiple clients, same port:** Works because each connection has a unique 5-tuple. The server port is the same, but each client has a different IP:port combination.

### Why a Server Can Handle Thousands of Connections on One Port

```
Connection 1: (192.168.1.10, 52431, 93.184.216.34, 443, TCP)
Connection 2: (192.168.1.10, 52432, 93.184.216.34, 443, TCP)  ← different src port
Connection 3: (10.0.0.5,     48891, 93.184.216.34, 443, TCP)  ← different src IP
```

All three connect to the same server IP:port (93.184.216.34:443), but each is a unique connection identified by the 5-tuple.

---

## Interview Questions + Answers

---

**Q1: What is a socket?**

**Ideal Answer:**
"A socket is an endpoint for network communication, identified by an IP address and port number. It's the interface between the application and the transport layer. A TCP connection is established between two sockets — one on the client and one on the server. The connection is uniquely identified by the 5-tuple: source IP, source port, destination IP, destination port, and protocol."

---

**Q2: What is the difference between a port and a socket?**

**Ideal Answer:**
"A port is just a 16-bit number that identifies a service or process. A socket is a combination of IP address + port — it's an actual endpoint for communication. A port is like an apartment number; a socket is the complete address (building + apartment) that you can actually send mail to."

---

**Q3: How can a server handle multiple clients on the same port?**

**Ideal Answer:**
"Because each connection is identified by the full 5-tuple: source IP, source port, destination IP, destination port, and protocol. While the server's IP and port are the same for all connections, each client has a different source IP and/or source port. When `accept()` is called, it creates a new socket for that specific client connection, while the original listening socket continues accepting new connections."

---

**Q4: Explain the socket lifecycle — bind, listen, accept, connect.**

**Ideal Answer:**
"On the server: `socket()` creates the socket, `bind()` attaches it to an IP and port, `listen()` makes it ready for connections, and `accept()` blocks until a client connects — then returns a new socket dedicated to that client. On the client: `socket()` creates the socket, `connect()` initiates the TCP handshake to the server. Both sides use `send()/recv()` for data and `close()` to terminate."

---

**Q5: What are ephemeral ports?**

**Ideal Answer:**
"Ephemeral ports are temporary port numbers (49152–65535) assigned by the OS to client-side connections. When a client connects to a server, the OS picks a random available ephemeral port as the source port. This allows multiple connections from the same machine to the same server, each with a different source port."

---

**Q6: What is the 5-tuple that identifies a connection?**

**Ideal Answer:**
"Source IP, source port, destination IP, destination port, and protocol. This combination uniquely identifies every TCP or UDP connection. It's how the OS knows which socket to deliver incoming data to."

---

**Q7: How do sockets relate to HTTP?**

**Ideal Answer:**
"HTTP is built on top of TCP sockets. When a browser makes an HTTP request, it creates a TCP socket, connects to the server (triggering the 3-way handshake), optionally does a TLS handshake, then sends the HTTP request as bytes over the socket and reads the response bytes back. HTTP libraries and web frameworks abstract this, but underneath it's all socket operations."

---

### Common Mistakes

| Mistake | Correction |
|---------|------------|
| "A server needs a different port for each client" | A server uses **one port** for all clients. Connections are distinguished by the **5-tuple** (different client IP:port combinations). |
| "accept() uses the listening socket for data transfer" | `accept()` returns a **new socket** for the client. The listening socket stays open for more connections. |
| "TCP port 80 and UDP port 80 are the same" | TCP and UDP have **independent** port spaces. TCP:80 and UDP:80 are different. |
| "The client chooses its own port" | The **OS assigns** an ephemeral port automatically. The client doesn't (normally) choose. |

---

### Interview Takeaways — Module 8

1. **Port = 16-bit number identifying a process.** Well-known (0–1023), registered (1024–49151), ephemeral (49152–65535).
2. **Socket = IP + Port.** It's the endpoint for communication.
3. **5-tuple** (src IP, src port, dst IP, dst port, protocol) uniquely identifies a connection.
4. **Server lifecycle:** socket() → bind() → listen() → accept() → read/write → close().
5. **Client lifecycle:** socket() → connect() → read/write → close().
6. **`accept()` returns a NEW socket** — the listening socket keeps accepting.
7. **One server port, thousands of clients** — works because each client has a unique source IP:port.
