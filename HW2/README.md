# HTTP Homework - Internet Engineering (Fall 1405)

**Name:** Roza Ganjipour

**Student ID:** 40131040

## Question 1

### Differences between HTTP versions

**HTTP/0.9 (1991)**
- Only the GET method
- No headers, no status codes
- Response is raw HTML only
- Connection closes after each response

**HTTP/1.0 (1996)**
- Request and response headers added
- Status codes (200, 404, ...)
- New methods: POST, HEAD
- Content-Type, so files other than HTML can be sent
- Version number in the request line
- Still one request per TCP connection (keep-alive existed but was not standard)

**HTTP/1.1 (1997)**
- Persistent connections by default
- Host header is mandatory, so many sites can share one IP (virtual hosting)
- Pipelining (rarely used in practice because of head-of-line blocking)
- Chunked transfer encoding
- Better caching: Cache-Control, ETag, If-None-Match
- New methods: PUT, DELETE, OPTIONS, TRACE
- Range requests (resuming downloads)
- Still text based, and responses on one connection come back in order

**HTTP/2 (2015)**
- Binary framing instead of text
- Multiplexing: many streams over one TCP connection
- Header compression (HPACK)
- Stream priorities
- Server push (later dropped by browsers)
- In practice used only over TLS
- Still on TCP, so one lost packet blocks every stream (TCP head-of-line blocking)

**HTTP/3 (2022)**
- Runs on QUIC, which runs on UDP
- TLS 1.3 is part of the transport, not a separate layer
- Faster setup: 1-RTT, and 0-RTT for repeat connections
- Streams are independent, a lost packet only stalls its own stream
- Connection migration: the connection has an ID, so switching from Wi-Fi to mobile data doesn't drop it
- Header compression with QPACK

### Why HTTP/3 uses UDP

The problems HTTP/2 still had came from TCP itself. TCP delivers bytes strictly in order, so if one packet is lost, data for all streams waits until it is retransmitted, even data that belongs to other requests. Fixing this would need changes to TCP, but TCP is implemented in operating system kernels and is inspected by routers, firewalls and NAT devices all over the internet. Changing it takes many years, and a brand new transport protocol (SCTP for example) gets dropped by a lot of these middleboxes.

UDP is very simple and is already allowed almost everywhere. It doesn't add ordering or reliability, so QUIC can build its own versions of these on top of it: per-stream reliability, its own congestion control, and encryption merged into the handshake. Because QUIC runs in user space (in the browser or server application), it can be updated as fast as any other software instead of waiting for OS updates. So UDP isn't chosen because it is "unreliable", it is chosen because it gets out of the way and lets QUIC do things TCP can't.

## Question 2

### a) Stateless vs stateful

In a stateless protocol the server keeps no memory of earlier requests, so every request must carry all the information needed to process it, and any server in a cluster can answer it. In a stateful protocol the server keeps session information between requests (for example FTP remembering your current directory, or an SSH session). Stateless systems are easier to scale and recover from crashes, while stateful systems are more convenient but tie the client to one server and cost memory for every open session.

### b) Where HTTP fits and how it deals with it

HTTP is stateless. Each request is handled on its own and the server doesn't know that two requests came from the same user. That is a problem for things like logins and shopping carts. The main ways HTTP handles it:

- **Cookies:** the server sends a `Set-Cookie` header, and the browser sends it back in a `Cookie` header on every later request to that site.
- **Server-side sessions:** the cookie only holds a random session ID, and the actual data (user ID, cart contents) is stored on the server, in memory or in something like Redis.
- **Tokens:** for example a JWT sent in the `Authorization: Bearer ...` header. The state is inside the token and is signed, so the server can check it without storing anything. This keeps the server stateless and is common in REST APIs and mobile apps.
- Older methods like putting a session ID in the URL or in hidden form fields. These are mostly avoided now because URLs leak through logs and the Referer header.

So the protocol stays stateless, and the state is added on top of it by the application.

## Question 3

### a) The challenge

In HTTP only the client can start a request, and the server can only answer. When something happens on the server side (a new chat message, a server alert, a payment confirmed), the server has no way to tell the client right away. The client has to keep asking "is there anything new?". If it asks often, most requests come back empty and waste bandwidth, battery and server resources. If it asks rarely, the user sees the data late. For a chat or monitoring system, neither is acceptable. On top of that, most clients sit behind NAT or firewalls, so the server can't open a connection to them even if it wanted to.

### b) Solutions

**Short polling.** The client sends a request every few seconds and the server answers immediately, with new data or with nothing. It is the simplest method and works everywhere, but there is always a delay of up to one polling interval and most requests are wasted. It is still fine for data that changes slowly, like checking the status of a long job.

**Long polling.** The client sends a request and the server doesn't answer until it has new data or a timeout passes (for example 30 seconds). When the client gets an answer, it immediately sends a new request. This way the data arrives almost instantly and empty responses are rare. The downside is that the server keeps many requests open at once, and each message still needs a new HTTP request. Older versions of Facebook chat worked like this.

**Server-Sent Events (SSE).** The client opens one HTTP request and the server keeps the response open, sending events as text (`Content-Type: text/event-stream`) whenever something happens. The browser has a built-in `EventSource` API that also reconnects automatically. It is one-way only (server to client), but for notifications, live scores, stock prices or streaming the output of an AI model that is all you need. It is plain HTTP, so proxies and load balancers usually handle it without problems.

**WebSocket.** The connection starts as an HTTP request with an `Upgrade: websocket` header, and after the server answers with `101 Switching Protocols` it becomes a full-duplex channel over the same TCP connection. Both sides can send messages at any time with very little overhead per message. This is the usual choice for chat apps, online games and collaborative editors. The cost is that the server must keep a stateful connection per user, which makes scaling and load balancing harder.

**Webhooks.** Here the roles are reversed. The service that has the event (say, a payment gateway or GitHub) acts as the client and sends an HTTP POST to a URL that the receiving side registered earlier. Nobody polls; the request is sent only when the event happens. This works between servers, not for browsers, because the receiver needs a public URL. The receiver should also verify the request, usually with an HMAC signature in a header, and the sender needs a retry policy for when the receiver is down.

**Push notification services.** Mobile apps can't keep a connection open all the time, so phones keep a single connection to the platform's service (Firebase Cloud Messaging on Android, APNs on iOS, Web Push for browsers). The app's server sends the message to that service and the service delivers it to the device, even when the app is closed.

(HTTP/2 server push is sometimes mentioned here, but it isn't a real solution: it could only push files the browser would ask for anyway, like CSS, not application events. Browsers have removed it.)

## Question 4

URL: `ss://asghar:1234!!@ss.myproxy.com:1234#shadowSocks1`

| Part | Value | Notes |
|---|---|---|
| Scheme | `ss` | Shadowsocks proxy protocol |
| User info | `asghar:1234!!` | Everything before `@` |
| Username | `asghar` | |
| Password | `1234!!` | `!` is allowed in user info without encoding |
| Host | `ss.myproxy.com` | `ss` is the subdomain, `myproxy` the domain, `com` the TLD |
| Port | `1234` | |
| Path | empty | |
| Query | none | there is no `?` |
| Fragment | `shadowSocks1` | Everything after `#` |

Some notes:
- The authority part is `asghar:1234!!@ss.myproxy.com:1234`, which is user info + host + port.
- The fragment is never sent to the server. In Shadowsocks links it is used as a display name (tag) for the server in the client app.
- The port number and the password happen to both be `1234`; they are separate fields.
- In real Shadowsocks links (SIP002 format) the user info is normally `method:password` encoded in Base64, not a plain username. Putting a password in a URL in plain text is also not recommended, since URLs get saved in history and logs.

## Question 5

1. **Database failure, logins don't work: `503 Service Unavailable`.** The problem is on the server side and is temporary, and 503 says exactly that. A `Retry-After` header can tell the client when to try again. `500 Internal Server Error` is also correct but more generic; I prefer 503 because it tells the client that retrying later makes sense.

2. **Username doesn't exist, login failed: `401 Unauthorized`.** The client failed to authenticate. `404 Not Found` might seem logical since the user doesn't exist, but it lets an attacker find out which usernames are registered (user enumeration). The safe choice is to return the same 401 and the same message ("invalid username or password") whether the username or the password was wrong.

3. **Website moved to another domain: `301 Moved Permanently` or `308 Permanent Redirect`**, with the new address in the `Location` header. Both are permanent, so clients and search engines update their links. For a REST API I would use 308, because with 301 some clients change a POST into a GET when following the redirect, while 308 guarantees the method and body stay the same. For normal web pages 301 is fine.

4. **Too many requests from this user: `429 Too Many Requests`.** This code was made for rate limiting. It is good practice to add `Retry-After` and headers like `RateLimit-Remaining` so the client knows how long to wait.

5. **Token refresh succeeded, new token in a header: `200 OK`.** The request worked and there is nothing special about it. Since the token is in a header and the body might be empty, `204 No Content` also fits. I wouldn't use `201 Created` because the token isn't a new resource with its own URL.

6. **User is from a banned (sanctioned) IP range: `403 Forbidden` or `451 Unavailable For Legal Reasons`.** 403 means the server understood the request but refuses it, which is true. But since the block is because of sanctions, which is a legal requirement and not a choice about this user's permissions, 451 is more accurate. It was defined exactly for content blocked because of legal demands. I would choose 451 because it tells the user the real reason. 401 would be wrong here, logging in wouldn't change anything.

## Question 6

1. **University servers instead of central servers for apt packages: forward proxy with caching (caching proxy).** The proxy sits on the client side, inside the university network. The first time a package is requested it is downloaded from the main server and saved; after that, other students get it from the local copy. This saves international bandwidth and is much faster. Examples: Squid, apt-cacher-ng. (If the university fully copies the repository ahead of time it is called a mirror, but the idea of serving content from a nearby cache is the same.)

2. **Splitting requests by geographic origin: reverse proxy used as a load balancer (geo-based load balancing).** The proxy sits in front of the servers. It looks at the client's IP address, finds its location, and sends the request to the nearest or the right data center. CDNs work this way, and it is often combined with GeoDNS.

3. **Adding authentication to a very old service: reverse proxy (authentication gateway).** The old service is hidden behind the proxy and not reachable directly. The proxy checks the user's credentials (Basic Auth, OAuth/OIDC, or a token) and forwards only authenticated requests to the old service, which doesn't need any changes. Examples: nginx with `auth_request`, oauth2-proxy, or an API gateway.

## Use of AI tools

I used AI to help articulate my answers more clearly and to come up with better examples.