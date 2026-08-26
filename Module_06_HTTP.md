# Module 6 — HTTP

> **HIGH PRIORITY MODULE**

---

## 1. What HTTP Is `MUST KNOW`

**HTTP = HyperText Transfer Protocol** — the application-layer protocol for web communication.

- **Client-server model:** Client (browser) sends a request, server processes and sends a response.
- Runs over **TCP** (HTTP/1.1, HTTP/2) or **QUIC/UDP** (HTTP/3).
- Default ports: **80** (HTTP), **443** (HTTPS).
- **Text-based** (HTTP/1.1) or **binary** (HTTP/2, HTTP/3).

---

## 2. HTTP Request and Response `MUST KNOW`

### HTTP Request

```
GET /api/users?page=1 HTTP/1.1        ← Request Line (Method, Path, Version)
Host: example.com                      ← Headers
Accept: application/json
Authorization: Bearer eyJ...
Cookie: session_id=abc123
User-Agent: Mozilla/5.0
                                       ← Empty line (separates headers from body)
                                       ← Body (empty for GET)
```

### HTTP Response

```
HTTP/1.1 200 OK                        ← Status Line (Version, Status Code, Reason)
Content-Type: application/json         ← Headers
Content-Length: 85
Set-Cookie: session_id=xyz789
Cache-Control: max-age=3600
                                       ← Empty line
{"users": [{"id": 1, "name": "Alice"}]} ← Body
```

---

## 3. HTTP Methods `MUST KNOW`

| Method | Purpose | Has Body? | Idempotent? | Safe? |
|--------|---------|-----------|-------------|-------|
| **GET** | Retrieve data | No | ✅ Yes | ✅ Yes |
| **POST** | Create / submit data | Yes | ❌ No | ❌ No |
| **PUT** | Replace entire resource | Yes | ✅ Yes | ❌ No |
| **PATCH** | Partial update | Yes | ❌ No | ❌ No |
| **DELETE** | Delete resource | Optional | ✅ Yes | ❌ No |

### Idempotency `MUST KNOW`
Making the **same request N times** has the **same effect as making it once**.

- **GET:** Reading the same resource 10 times → same result. ✅ Idempotent.
- **PUT:** Replacing a resource with the same data 10 times → same result. ✅ Idempotent.
- **DELETE:** Deleting a resource 10 times → resource is deleted (subsequent calls return 404, but the state is the same). ✅ Idempotent.
- **POST:** Creating a resource 10 times → 10 resources created. ❌ NOT idempotent.

### Safe Methods
**Don't modify server state.** Only **GET** (and HEAD, OPTIONS) are safe. You can call them without side effects.

### Key Comparisons

**GET vs POST:**
| | GET | POST |
|-|-----|------|
| Purpose | Retrieve data | Create/submit data |
| Body | No body | Has body |
| Idempotent | Yes | No |
| Cacheable | Yes | Generally no |
| Bookmarkable/URL | Parameters in URL | Parameters in body |

**PUT vs PATCH:**
| | PUT | PATCH |
|-|-----|-------|
| Purpose | Replace entire resource | Partial update |
| Idempotent | Yes | No (in general) |
| Body | Full resource representation | Only changed fields |

---

## 4. HTTP Status Codes `MUST KNOW`

### By Category

| Range | Meaning |
|-------|---------|
| **1xx** | Informational |
| **2xx** | Success |
| **3xx** | Redirection |
| **4xx** | Client Error |
| **5xx** | Server Error |

### Must-Know Codes

| Code | Meaning | Interview Detail |
|------|---------|-----------------|
| **200** | OK | Request succeeded. Standard success response. |
| **201** | Created | Resource successfully created (POST). |
| **204** | No Content | Success but no response body (DELETE). |
| **301** | Moved Permanently | Resource permanently at new URL. Browsers cache this. **SEO:** link equity transfers. |
| **302** | Found (Temporary Redirect) | Resource temporarily at different URL. Don't cache. |
| **304** | Not Modified | Resource hasn't changed — use cached version. Returned with ETag/If-None-Match. |
| **400** | Bad Request | Malformed request syntax. Client sent invalid data. |
| **401** | Unauthorized | **Authentication** required. "Who are you?" — provide credentials. |
| **403** | Forbidden | **Authorization** failed. "I know who you are, but you don't have permission." |
| **404** | Not Found | Resource doesn't exist at this URL. |
| **405** | Method Not Allowed | HTTP method not supported for this endpoint (e.g., POST to a GET-only endpoint). |
| **429** | Too Many Requests | Rate limiting. Client sent too many requests. |
| **500** | Internal Server Error | Server crashed / unhandled exception. Generic server failure. |
| **502** | Bad Gateway | Reverse proxy/load balancer received invalid response from upstream server. |
| **503** | Service Unavailable | Server temporarily overloaded or in maintenance. |
| **504** | Gateway Timeout | Reverse proxy/load balancer timed out waiting for upstream server. |

**Critical distinction — 401 vs 403:**
- **401:** "I don't know who you are. Please authenticate." (Missing or invalid credentials.)
- **403:** "I know who you are, but you can't access this." (Authenticated but not authorized.)

---

## 5. Statelessness, Cookies, Sessions `MUST KNOW`

### HTTP is Stateless
Each request is **completely independent**. The server doesn't remember previous requests. Request #2 knows nothing about request #1.

**Why stateless?**
- **Simplicity:** Server doesn't need to track per-client state.
- **Scalability:** Any server can handle any request (critical for load balancing).
- **Reliability:** No state to lose if a server crashes.

### How State is Maintained

#### Cookies
1. Server sends `Set-Cookie: session_id=abc123` in the response header.
2. Browser stores the cookie.
3. Browser automatically sends `Cookie: session_id=abc123` with every subsequent request to that domain.
4. Server reads the cookie to identify the client.

#### Sessions
- **Server-side storage** indexed by a session ID.
- The session ID is stored in a cookie.
- Server maintains the actual state (user info, cart, preferences) in memory or a database.
- Flow: Authenticate → server creates session → session ID sent as cookie → subsequent requests include cookie → server looks up session.

**Interview answer:** "HTTP is stateless by design for simplicity and scalability. State is maintained using cookies — the server sends a Set-Cookie header, the browser stores it and sends it with every subsequent request. Sessions are server-side storage keyed by a session ID that's carried in a cookie. This is how logins persist — the cookie carries the session ID, the server looks up the user's state."

---

## 6. HTTP Caching `GOOD TO KNOW`

### Cache-Control Header

| Directive | Meaning |
|-----------|---------|
| `max-age=3600` | Cache for 3600 seconds |
| `no-cache` | Must revalidate with server before using cached version |
| `no-store` | Don't cache at all (sensitive data) |
| `public` | Any cache (CDN, proxy) can store it |
| `private` | Only browser can cache (not CDN/proxy) |

### ETag (Entity Tag)
- Server sends `ETag: "abc123"` (hash of the resource).
- Next request: client sends `If-None-Match: "abc123"`.
- If resource unchanged → server returns **304 Not Modified** (no body — saves bandwidth).
- If changed → server returns **200** with new content and new ETag.

**Why caching matters:** Reduces latency, server load, and bandwidth usage.

---

## 7. Keep-Alive / Persistent Connections `MUST KNOW`

- **HTTP/1.0:** New TCP connection for every request. Expensive — 1 RTT for each TCP handshake.
- **HTTP/1.1:** **Persistent connections** by default (`Connection: keep-alive`). Multiple requests reuse the same TCP connection.

**Why this matters:** A web page makes dozens of requests (HTML, CSS, JS, images). Opening a new TCP connection (+ TLS handshake for HTTPS) for each one wastes 2-3 RTTs per request. Persistent connections amortize this cost.

---

## 8. HTTP Versions `MUST KNOW`

### HTTP/1.1
- **Persistent connections** (keep-alive) — default.
- **Pipelining** — send multiple requests without waiting for responses (in theory; rarely used in practice).
- **Head-of-line (HOL) blocking:** Responses must come back in order. A slow response blocks all subsequent responses on that connection.
- **Workaround:** Browsers open **6 parallel TCP connections** per domain.

### HTTP/2
- **Binary framing layer** — data is encoded in binary frames (not text), more efficient to parse.
- **Multiplexing:** Multiple requests and responses over a **single TCP connection**, interleaved as frames in independent **streams**. No HTTP-level HOL blocking.
- **Streams:** Each request/response pair is a stream with a unique ID. Frames from different streams can be interleaved.
- **Header compression (HPACK):** Reduces overhead of repeated headers (cookies, user-agent, host are sent with every request in HTTP/1.1).
- **Server push:** Server proactively sends resources it thinks the client will need (less used in practice).

**Why HTTP/2 is faster:**
- Multiplexing eliminates HTTP-level HOL blocking.
- Single connection = 1 TCP handshake + 1 TLS handshake (not 6).
- Binary format is more efficient.
- Header compression reduces bandwidth.

**BUT:** Still has **TCP-level HOL blocking** — if a TCP segment is lost, ALL streams are blocked until it's retransmitted (because TCP guarantees ordered delivery).

### HTTP/3
- Runs over **QUIC** (which runs over **UDP**, not TCP).
- QUIC provides: reliability, encryption (TLS 1.3 built-in), multiplexing with **independent streams**.
- **No TCP-level HOL blocking:** If a packet for stream A is lost, streams B and C are unaffected. QUIC handles per-stream retransmission.
- **Faster connection setup:** QUIC combines transport + TLS handshake into **1 RTT** (or **0-RTT** for returning clients).
- **Connection migration:** QUIC connections survive IP changes (e.g., switching from Wi-Fi to cellular).

**Why HTTP/3 exists:** To solve TCP's HOL blocking problem, which HTTP/2 inherited.

### HTTP Version Comparison

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------|----------|--------|--------|
| **Transport** | TCP | TCP | QUIC (over UDP) |
| **Format** | Text | Binary | Binary |
| **Multiplexing** | No (1 request at a time per connection) | Yes (streams over 1 TCP connection) | Yes (independent streams over QUIC) |
| **HOL Blocking** | Yes (HTTP level) | Solved at HTTP level, but TCP-level HOL remains | Solved at both levels |
| **Header Compression** | No | HPACK | QPACK |
| **Encryption** | Optional | Practically required (TLS) | Mandatory (TLS 1.3 built-in) |
| **Connection Setup** | TCP + TLS = 2-3 RTT | TCP + TLS = 2-3 RTT | 1 RTT (0-RTT for resumption) |
| **Connection Migration** | No | No | Yes |

---

## Interview Questions + Answers

---

**Q1: What is HTTP?**

**Ideal Answer:**
"HTTP is the HyperText Transfer Protocol — an application-layer protocol for client-server web communication. The client sends a request with a method (GET, POST, etc.), headers, and optional body. The server responds with a status code, headers, and body. HTTP is stateless — each request is independent. It runs over TCP (HTTP/1.1, HTTP/2) or QUIC/UDP (HTTP/3)."

---

**Q2: Explain the HTTP request-response structure.**

**Ideal Answer:**
"An HTTP request has a request line (method, URL, version), headers (Host, Content-Type, Authorization, etc.), and an optional body. An HTTP response has a status line (version, status code, reason), headers (Content-Type, Set-Cookie, Cache-Control), and a body. An empty line separates headers from the body in both."

---

**Q3: What are the main HTTP methods? Explain idempotency.**

**Ideal Answer:**
"GET retrieves data, POST creates/submits, PUT replaces entirely, PATCH partially updates, DELETE removes. Idempotency means making the same request multiple times has the same effect as making it once. GET, PUT, and DELETE are idempotent. POST is not — each POST can create a new resource."

---

**Q4: GET vs POST?**

**Ideal Answer:**
"GET retrieves data with no body — parameters go in the URL, it's cacheable and idempotent. POST submits data in the request body, it's not cacheable and not idempotent. GET is safe (no side effects); POST modifies server state. Use GET for reading data, POST for creating resources or submitting forms."

---

**Q5: PUT vs PATCH?**

**Ideal Answer:**
"PUT replaces the entire resource — you send the complete representation. PATCH updates only specific fields — you send just the changes. PUT is idempotent (replacing with the same data N times = same result). PATCH is generally not idempotent."

---

**Q6: What does 401 vs 403 mean?**

**Ideal Answer:**
"401 Unauthorized means authentication is required — the client didn't provide valid credentials. It's a 'who are you?' error. 403 Forbidden means the client is authenticated but not authorized — 'I know who you are, but you don't have permission.' 401 is about identity; 403 is about permissions."

---

**Q7: What is the difference between 301 and 302?**

**Ideal Answer:**
"301 is a permanent redirect — the resource has permanently moved to a new URL. Browsers and search engines cache this. 302 is a temporary redirect — the resource is temporarily at a different URL. The original URL should still be used for future requests. For SEO, 301 transfers link equity; 302 does not."

---

**Q8: Why is HTTP stateless? How do cookies and sessions maintain state?**

**Ideal Answer:**
"HTTP is stateless for simplicity and scalability — any server can handle any request, which is essential for load balancing. State is maintained using cookies: the server sends a Set-Cookie header, the browser stores it and includes it in every subsequent request. For sessions, the server stores state server-side indexed by a session ID, which is carried in a cookie. This is how login persistence works."

---

**Q9: What is HTTP/2 and why is it faster than HTTP/1.1?**

**Ideal Answer:**
"HTTP/2 uses binary framing and multiplexing — multiple requests and responses flow over a single TCP connection as independent streams. This eliminates HTTP-level head-of-line blocking where a slow response in HTTP/1.1 blocks all others. It also uses HPACK header compression to reduce overhead. The downside is it still has TCP-level HOL blocking — a single lost TCP segment blocks all streams."

> **Follow-up Q: What is head-of-line blocking?**
>
> **Ideal Answer:** "In HTTP/1.1, responses on a connection must come in order. If the first response is slow, all subsequent responses are blocked. HTTP/2 solves this at the HTTP level with multiplexed streams, but TCP still delivers bytes in order, so a lost TCP packet blocks all streams. HTTP/3 solves this completely by using QUIC over UDP with independent per-stream delivery."

---

**Q10: What is HTTP/3? Why does it use QUIC?**

**Ideal Answer:**
"HTTP/3 runs over QUIC, which is built on UDP instead of TCP. QUIC provides reliability, encryption (TLS 1.3 built-in), and multiplexing with truly independent streams — a lost packet for one stream doesn't block others. It also has faster connection setup (1-RTT, or 0-RTT for returning clients) and supports connection migration (survives IP changes). HTTP/3 exists to solve the TCP-level HOL blocking that HTTP/2 couldn't fix."

---

**Q11: How does HTTP caching work?**

**Ideal Answer:**
"The server uses Cache-Control headers to specify caching rules — max-age for duration, no-store to prevent caching. ETags provide validation: the server sends a hash of the resource, and the client sends it back on the next request. If the resource hasn't changed, the server returns 304 Not Modified, saving bandwidth. Caching reduces latency, server load, and bandwidth."

---

**Q12: What is the difference between HTTP and HTTPS?**

**Ideal Answer:**
"HTTPS is HTTP over TLS. The difference is that HTTPS encrypts all communication between client and server using TLS, providing confidentiality (data can't be read), integrity (data can't be modified), and authentication (you're talking to the real server). HTTP sends everything in plaintext. HTTPS uses port 443; HTTP uses port 80."

---

### Common Mistakes

| Mistake | Correction |
|---------|------------|
| "POST is idempotent" | POST is **NOT** idempotent — each POST can create a new resource. PUT and DELETE are idempotent. |
| "401 means you don't have permission" | 401 = **authentication** issue ("who are you?"). 403 = **authorization** issue ("you can't access this"). |
| "HTTP/2 fixes all head-of-line blocking" | HTTP/2 fixes HTTP-level HOL blocking. **TCP-level** HOL blocking remains. HTTP/3 (QUIC/UDP) fixes both. |
| "HTTP is secure enough" | HTTP is **plaintext**. Anyone on the network can read/modify the data. HTTPS (TLS) is required for security. |
| "GET requests can't be cached" | GET is **the most cacheable** method. POST is generally not cached. |
| "HTTP/3 uses TCP" | HTTP/3 uses **QUIC over UDP**. That's the whole point — to escape TCP's limitations. |

---

### Interview Takeaways — Module 6

1. **HTTP = request/response protocol.** Request: method + URL + headers + body. Response: status code + headers + body.
2. **Methods:** GET (read), POST (create), PUT (replace), PATCH (partial update), DELETE (remove).
3. **Idempotent:** GET, PUT, DELETE. **NOT idempotent:** POST. **Safe:** only GET.
4. **Status codes:** 2xx = success, 3xx = redirect, 4xx = client error, 5xx = server error.
5. **401 = authenticate yourself. 403 = you're authenticated but not authorized.**
6. **HTTP is stateless.** Cookies carry state. Sessions store state server-side with session ID in a cookie.
7. **HTTP/1.1:** Persistent connections, but HOL blocking.
8. **HTTP/2:** Binary, multiplexed streams over one TCP connection. Eliminates HTTP-level HOL blocking.
9. **HTTP/3:** QUIC over UDP. Independent streams. No HOL blocking at any level. 1-RTT setup.
10. **ETag + 304 Not Modified:** Efficient cache validation.
11. **Keep-alive** reuses TCP connections — avoids repeated handshake overhead.
12. **301 = permanent redirect (cached). 302 = temporary redirect (not cached).**
