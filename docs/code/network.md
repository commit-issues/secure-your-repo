# Network Security & Data Integrity

*Every request your application makes is a potential interception point.*

---

!!! info "TL;DR"

    - Every network call your application makes travels through infrastructure you do not own or control — servers, routers, ISPs, and third-party vendors, any of which can be a weak link in the chain.
    - SSL and TLS are the encryption protocols that protect data in transit. Understanding them is the foundation for everything else on this page.
    - Without HTTPS, everything you send is readable by anyone on the path — credentials, tokens, user data.
    - `verify=False` in Python disables SSL verification entirely. It is never acceptable in production code.
    - Every outbound request needs a timeout. A missing timeout is a denial-of-service vector waiting to be triggered.
    - CORS misconfiguration and missing security headers are among the most common and most avoidable vulnerabilities in web applications.

    Already comfortable with networking basics? Jump to [The Request Security Checklist →](#the-request-security-checklist)

---

You spent weeks securing your application. Input validation. Encrypted secrets. Signed commits. Dependency audits. Everything locked down.

Then your application made a network request — and everything you just protected left your controlled environment and traveled across infrastructure owned by strangers, maintained by organizations you have never audited, through systems that have been compromised before and will be again.

At some point today, your application will send data across a network. It will not go directly from your machine to its destination. It will hop through routers, pass through ISPs, cross data centers, touch third-party infrastructure — none of which you own, none of which you can inspect, any of which can be a weak link.

Most developers never think about this. The request goes out, the response comes back, and everything in between is invisible. That invisibility is exactly what attackers count on.

---

You saw in the [Dependency Security →](dependencies.md) section how a trusted package in your own supply chain can become the attack vector. The network is the same problem at a different layer — infrastructure you depend on, that you did not build, that you cannot fully control. This section covers what that actually means, why it matters for security, and exactly what to do about it. No networking background required — we start from the ground up.

---

## How Networks Actually Work

Before you can secure something, you need to understand what it is. Most security mistakes in network code happen because developers treat network calls as magic — data goes in, data comes out, and everything in between is someone else's problem. It is not.

### What a network actually is

Picture a city. It has buildings of all sizes — houses, offices, warehouses, data centers. It has roads connecting them. It has intersections managed by traffic systems, highways that carry long-distance traffic, and local streets that get you to a specific address.

Every building in this city can send and receive deliveries. To send something, you put it in a vehicle, give the driver a destination address, and send it out onto the road network. The vehicle doesn't travel in a straight line from your door to the destination — it goes through intersections, onto highways, through other neighborhoods, managed by road infrastructure that belongs to the city, not to you.

The internet works exactly like this. Every device connected to it — your laptop, a server in a data center in another country, the API your application calls, the payment processor your users trust — is a building in this city. The roads between them are the cables, fiber lines, and wireless connections that carry data from one place to another.

In networking, each of these connected devices is called a **node** — just a point on the network that can send or receive data. Your laptop is a node. The server running your application is a node. The router in your home is a node. A third-party API your code calls is a node. Every building in the city is a node on the network.

The internet is the largest version of this city — billions of nodes connected by a shared road system, all using the same rules for how to navigate it.

### What a network call is — and why your application makes them

Now that you can picture the city, the next question is: what is your application actually doing in it? Apps, websites, SaaS tools, internal platforms — they are all constantly sending and receiving information to function. For the customer placing an order, the student submitting an assignment, the business processing a transaction — every piece of that information travels along those digital roads. And every trip it makes is a potential exposure point.

Every action requires a network call. A **network call** is any time your application sends a vehicle out onto those roads to communicate. Your application does not sit in isolation processing data quietly. It is constantly in traffic.

It sends requests to external APIs to get authentication tokens, exchange rates, or user data. It talks to a database running in a different building on the other side of the city. It dispatches notifications through a third-party service, fetches configuration from cloud storage, or checks for software updates. Every one of those is a network call — your application sending something out across roads it does not own, to buildings it does not control, and waiting for a response to come back.

Each of those trips is an opportunity for something to go wrong — not because your code is broken, but because the roads and intersections between your building and the destination belong to other people. ISPs, cloud providers, data center operators, CDN vendors, and sometimes attackers who have positioned themselves somewhere along the route. The infrastructure in between is the part most developers never think about, until something goes wrong.

### APIs — what they are and where they fit

You will make network calls to APIs constantly. An **API** (Application Programming Interface) is an agreement for how two services communicate. Think of it like a courier delivery — there is a service window on the side of a building: you approach, show your credentials, hand over the package, request confirmation in the correct format, and the receiver hands back a response — a signed confirmation of receipt, and a return package. The courier has done his job according to protocol. He does not need to know what happens inside.

When you use an app, submit a form, or trigger any action — you are the sender. You handed off the package. You have no idea how it gets there. In the same way you do not know which route the bike courier took, you have no control over the roads your data travels. And just like anyone commuting through the city, the courier faces risks the entire time he is on the road — his bag can be gone through, the package tampered with, the contents swapped out, or the delivery rerouted entirely without him realizing it. If someone intercepts the package mid-route, you could receive something empty, something wrong, or something that looks exactly right but isn't. You would not know until it was too late.

This is where the supply chain problem from the [Dependency Security →](dependencies.md) section extends into the network layer. When the Axios npm package was compromised in March 2026, every API call that affected application made passed through compromised code. Imagine the courier delivered the package as expected — response received, everything looked normal. But inside that package was a hidden listening device, a tracker slipped in by a competitor, quietly recording every conversation and reporting back the whole time — and nobody knew it was there. The courier did his job, but after the package arrived, information was being stolen and the sender's building was being infiltrated through the very delivery service they trusted. Authentication headers, request bodies, API keys — all of it was readable to an attacker. Securing your network calls is not a separate concern from securing your supply chain. Your attack surface is constant and wide — you can be compromised from multiple directions at any time, and none of them announce themselves.

### How data actually travels — packets

Here is something that surprises most people: data does not travel across a network in one piece. When your application sends a request, that data gets broken into small chunks called **packets**. Think of it like shipping a large piece of furniture. Rather than trying to fit the whole sofa through the door at once, you disassemble it — cushions in one box, frame in another, legs in a third. Each box gets its own label with the destination address and a number indicating where it fits in the whole. They go out in separate vehicles, possibly taking completely different routes through the city, and get reassembled in the correct order when they all arrive at the destination.

Network packets work the same way. Each one carries a fragment of your data plus addressing information — where it came from, where it is going, and its sequence number so it can be reassembled correctly. The packets travel independently, may take different roads, and are put back together at the destination.

Why does this matter for security? Because an attacker who intercepts network traffic does not need to capture an entire connection. Individual packets carry real data. On an unencrypted connection — one where the contents are not scrambled — those packets are readable by anyone who can see them: credentials, API keys, user records, session tokens. All of it travels in packets, and without encryption, all of it is exposed. Encryption is what makes intercepted packets useless. We cover exactly how that works in the next section.

### IP addresses — the addressing system that makes delivery possible

Every device on a network has an IP address. Every server. Every router. Every phone. Every website. An IP address is how the network knows where to deliver packets.

For a packet to travel from your application to a server and back, the road network needs to know where everything is. Every building in our city has a unique street address. In networking, that address is called an **IP address**.

!!! info "How IP addresses work"

    Every device connected to the internet has a unique numerical address — its IP address. This is how the network knows where to deliver packets, the same way a street address tells a delivery driver where to go.

    IP addresses look like this: `203.0.113.42` — four numbers separated by dots, each between 0 and 255. Your laptop has one assigned by your router. The server running your application has one. The API you call has one. Even the DNS server we will cover shortly has one (`8.8.8.8` for Google's public resolver).

    When you write `mybank.com` in your code, your computer does not know the street address for that building yet. It has to look it up first — like checking a city directory for a business's address before you can send a vehicle there. That lookup is DNS, covered below.

### Ports — how one server handles many different services at once

A single server at a single IP address might be running a web server, a database, an email service, and an API all at the same time. Ports are how it keeps those services completely separate — and how your application knows exactly which one to talk to.

If the IP address is the building, the port is the apartment number inside it. One building, many apartments, each serving a completely different purpose, each with its own door.

Not all ports are protected the same way — and that is where the security issue comes in.

**Port 80** is where unencrypted HTTP traffic goes. No encryption, no verification. Everything sent through port 80 travels exactly as written — readable by anyone who intercepts it. Port 80 is like a building with no locks on the doors. Anyone walking by can walk in, look around, and read whatever is sitting on the table. You did not invite them. You did not know they were there.

**Port 443** is where HTTPS traffic goes — encrypted, verified, with identity checks before anything gets through. Port 443 is like a building with a deadbolt, a code lock on the entrance, and a system that verifies who is allowed in before they can proceed. Even legitimate access is scoped — like a dog walker who has the door code specifically to walk the dog, for that purpose, during those hours. A random person off the street cannot walk in and read what is on the table, because the system does not let them through the door in the first place.

A port **listens** for incoming connections. Listening means the service on that port is waiting — like a dedicated receptionist at a specific desk, ready to handle anyone who approaches that desk for that specific purpose. Port 443's receptionist handles encrypted HTTPS connections. Port 5432's receptionist handles PostgreSQL database connections. Each desk handles its own type of visitor and nobody else's.

When you leave a port open that should be closed, or run a service on a port without authentication, you have left a desk staffed with no one checking who sits down. The internet runs automated scanners around the clock, checking IP addresses for open ports — the same way someone might walk an entire street trying every door handle. An open, unprotected port will be found. The question is only when.

### DNS — the city directory

Every building has a street address. But most people do not navigate by address numbers alone — they look up businesses by name. "I need to get to the main office of Example Corp" — you search the city directory, find their address, then navigate there.

**DNS** — the Domain Name System — is that directory for the internet. When your application needs to reach `mybank.com`, it does not know the street address yet. It sends a query to a DNS server: "What is the IP address for `mybank.com`?" The DNS server checks the directory and sends back the answer. Only then can your application's vehicle be dispatched to the right building.

That lookup happens before your actual request goes anywhere — it is the step of checking the directory before leaving the building. And because it travels across the same road network, it can be intercepted.

An attacker who can tamper with DNS responses can give your application the wrong address — directing your vehicle to a building they control instead of the real destination. Your application loads up its cargo, locks the doors, and drives straight into the wrong building thinking it has arrived safely. We cover what that means and how to defend against it in the DNS Security section.

!!! note "A common misconception — IP addresses and your location"

    In this analogy we use street addresses to explain IP addresses, which makes sense for understanding how data gets routed. But in real life, an IP address does not pinpoint your exact location — and this is worth understanding clearly because a lot of people are either overly worried or not worried enough about it.

    Your IP address is assigned by your ISP — your internet provider. It identifies the ISP and the general region they serve, not your home. Think of it less like a street address and more like knowing someone lives somewhere in a particular zip code or neighborhood. Law enforcement can subpoena your ISP and request the account associated with a specific IP address at a specific time — that is how IP addresses are used to identify individuals legally. The ISP knows which customer was assigned that IP at that moment, and they can hand that over under a valid legal order. That is how people get caught.

    A VPN masks your real IP by routing your traffic through a server in a different location — so websites and services see the VPN server's IP, not yours. This is why people use VPNs for privacy. It does not make you invisible — the VPN provider now has your real IP instead — but it does add a layer of separation. What your IP leaking actually exposes is your ISP, a rough geographic area, and a thread that law enforcement or a determined attacker can pull on. That is worth protecting. It is just not the exact home address most people fear it is.

---

## The Threat in Plain Terms

Every request your application sends leaves your machine, travels through infrastructure you do not own, and arrives at its destination through a path you cannot see. A DNS lookup decides where it goes before you send it. Packets carry the actual data. Any node on the path can see unencrypted packets in full.

On a public network — a coffee shop, a conference, an airport — other people share the same network. Corporate proxies inspect traffic. ISPs log it. Compromised routers reroute it. Third-party vendors sit between you and the services you depend on.

Encryption is what makes all of that irrelevant. An attacker on the path who intercepts encrypted traffic sees ciphertext — useless noise without the keys to decrypt it. That encryption layer is called TLS, and it is what everything in the rest of this section is built on.

---

## SSL and TLS — The Encryption Layer

Before we go further, we need to establish what SSL and TLS actually are — because they appear throughout this entire section and most explanations skip past them entirely.

### What they are

**TLS** (Transport Layer Security) is the protocol that encrypts network connections. When your application connects to a server over HTTPS, TLS is doing the actual encryption. It scrambles everything being sent so that every node on the path between you and the server — routers, ISPs, intermediate infrastructure — sees only unreadable noise. Think of the `S` in `https://` as secured.

**SSL** (Secure Sockets Layer) is the older name. SSL was the original protocol, but it had security vulnerabilities and has been fully replaced by TLS. When people say "SSL" today, they almost always mean TLS. The terms are used interchangeably across documentation, tools, and error messages — including Python's `verify=False` flag and the built-in `ssl` library. If you see SSL mentioned anywhere, assume TLS is what is actually running.

### What TLS actually does

TLS does two things every time your application opens a connection to a server:

**1. It encrypts the connection.** Every byte sent between your application and the server is encrypted before it leaves your machine and can only be decrypted at the destination. No node in between can see it.

**2. It verifies identity.** TLS checks that the server you connected to is actually who it claims to be — not an impersonator who intercepted the connection. This is done using certificates, which we cover in the very next section.

### TLS versions — why the version matters

Not all TLS versions are equal. Older versions have known vulnerabilities and should not be in use anywhere.

```
TLS 1.0 — deprecated, do not use
TLS 1.1 — deprecated, do not use
TLS 1.2 — acceptable, widely supported, minimum acceptable standard
TLS 1.3 — current standard, faster and more secure than 1.2
```

Modern libraries default to TLS 1.2 or higher. If you are configuring your own server, explicitly disable older versions:

```python
# Python — enforce minimum TLS version
import ssl

context = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
context.minimum_version = ssl.TLSVersion.TLSv1_2
```

```javascript
// Node — enforce minimum TLS version
const https = require('https');

const server = https.createServer({
  minVersion: 'TLSv1.2',
});
```

TLS encrypts the connection. Certificates are how TLS proves the server on the other end is legitimate — and that is what we cover next.

---

## Certificates — What They Are and Why They Matter

A certificate is a verified identity document for a server — the digital equivalent of a passport or a business license. When your application connects to `https://mybank.com`, that server presents a certificate that says: "I am `mybank.com`, and a trusted authority has checked and signed off on this."

That trusted authority is called a **Certificate Authority (CA)** — an organisation that has been vetted and included in your operating system's list of entities it trusts. Apple, Google, Mozilla, and Microsoft all maintain these lists. When a CA issues a certificate to a server, they are cryptographically signing it — staking their verified reputation that this server is who it claims to be.

!!! info "What cryptographic signing means"

    Cryptographic signing is a way of proving that something is authentic and has not been tampered with. Think of it like a wax seal on a letter — except mathematically impossible to fake. When a CA signs a certificate, they use a private key that only they hold to generate a unique signature. Your operating system can verify that signature using the CA's public key. If the signature checks out, the certificate is genuine. If anyone tampered with it, the signature breaks and verification fails.

When your application verifies a certificate, it checks three things:

```
1. Was this certificate signed by a CA my system trusts?
2. Does the name on the certificate match the domain I am connecting to?
3. Has the certificate expired?
```

If all three pass, the connection proceeds. If any fail, the connection should be rejected — because something is wrong. Either the certificate is invalid, the domain does not match, or the server is not who it claims to be.

### Certificate chains

Certificates are not always signed directly by a root CA. Often there is a chain: a root CA signs an intermediate CA, which signs your server's certificate. TLS traces that chain back to a trusted root. This is called a certificate chain, and it is why you sometimes see errors mentioning intermediate certificates when something in the chain is misconfigured or missing.

### Certificate pinning

Standard certificate verification accepts any valid certificate for a domain, as long as it was signed by a trusted CA. Certificate pinning goes further: your application hardcodes the specific certificate — or its public key — that it expects to see. Even a valid, properly signed certificate from a legitimate CA will be rejected if it does not match the pinned value.

This protects against scenarios where a CA is compromised and fraudulently issues a certificate for a domain they do not control. It has happened to real CAs.

```python
# Python — basic certificate fingerprint verification
import hashlib
import ssl
import socket
import base64

def verify_cert_fingerprint(hostname, expected_fingerprint, port=443):
    context = ssl.create_default_context()
    with socket.create_connection((hostname, port)) as sock:
        with context.wrap_socket(sock, server_hostname=hostname) as ssock:
            cert_der = ssock.getpeercert(binary_form=True)
            actual = base64.b64encode(hashlib.sha256(cert_der).digest()).decode()

    if actual != expected_fingerprint:
        raise ValueError(
            f"Certificate fingerprint mismatch on {hostname}.\n"
            f"Expected: {expected_fingerprint}\n"
            f"Actual:   {actual}\n"
            "This may indicate a man-in-the-middle attack."
        )
```

**When pinning makes sense:**

- Mobile applications communicating with your own backend
- High-security internal services where you control both ends
- Situations where a CA compromise would have catastrophic consequences

**When it does not:**

- Calling third-party APIs — providers rotate certificates regularly, and pinning breaks on every rotation
- Anywhere you do not control the certificate rotation schedule

### Certificate expiry

Certificates expire. When they do, your application's connections fail — or worse, if verification is disabled, they silently continue to an unverified destination. Monitor your certificates. Most providers send expiry warnings, but do not rely solely on that. Set up your own alerts with enough lead time to renew without disruption. A certificate expiry that takes down production at 2am is entirely preventable.

!!! warning "Vibe coding and hardcoded values"

    AI-generated code frequently hardcodes sensitive values — certificate paths, expiry dates, secrets, and credentials — directly into source files where they do not belong. If you are using AI to help write or scaffold your code, audit every value it hardcodes. Certificate paths, private key locations, and any environment-specific configuration should never be hardcoded. They belong in environment variables or a secrets manager — never in your source code. See [Credential Management →](credentials.md) and [Vibe Coding & AI Dev →](vibe-coding.md) for how to handle this correctly.

---

## HTTPS — Not Optional

HTTP is the protocol applications use to communicate with servers. HTTPS is HTTP with TLS encryption layered on top — the same TLS we just covered. That encryption is what prevents the nodes between your application and the server from reading your traffic.

Without HTTPS, every request is plaintext. Your credentials, your API keys, your users' data — readable by any node on the path.

### What the padlock actually means

When a browser shows a padlock icon, it means the connection is encrypted using TLS. It does not mean the site is trustworthy or that the operator is legitimate. A phishing site can have a padlock. A malicious service can serve over HTTPS.

The padlock tells you the connection is private. The certificate tells you the server is who it claims to be. Both matter — one without the other is incomplete.

### Enforcing HTTPS in your application

Never use `http://` URLs when `https://` is available:

```python
# Python — requests

# Wrong — sends everything across every node in plaintext
response = requests.get("http://mybank.com/data")

# Correct
response = requests.get("https://mybank.com/data")
```

```javascript
// Node — fetch

// Wrong
const response = await fetch("http://mybank.com/data");

// Correct
const response = await fetch("https://mybank.com/data");
```

If you are building a web server, redirect all HTTP traffic to HTTPS at the application level. Never let an unencrypted connection through:

```python
# Flask — redirect HTTP to HTTPS
from flask import Flask, redirect, request

app = Flask(__name__)

@app.before_request
def enforce_https():
    if not request.is_secure:
        return redirect(request.url.replace("http://", "https://"), code=301)
```

```javascript
// Express — redirect HTTP to HTTPS
app.use((req, res, next) => {
  if (!req.secure) {
    return res.redirect(301, `https://${req.headers.host}${req.url}`);
  }
  next();
});
```

### HSTS — making the enforcement permanent

Even with server-side HTTPS enforcement, a user or client could still attempt an initial plain HTTP connection before being redirected. HSTS (HTTP Strict Transport Security) closes that window.

HSTS is a response header that tells browsers: never connect to this domain over plain HTTP again, even if someone types `http://` or follows an unencrypted link. The browser remembers this instruction and upgrades all future connections automatically — before they leave the device.

```python
# Flask
@app.after_request
def set_hsts(response):
    response.headers['Strict-Transport-Security'] = (
        'max-age=31536000; includeSubDomains'
    )
    return response
```

```javascript
// Express
app.use((req, res, next) => {
  res.setHeader(
    'Strict-Transport-Security',
    'max-age=31536000; includeSubDomains'
  );
  next();
});
```

`max-age=31536000` is one year in seconds. Start with a shorter value during testing and extend once you are confident everything in your application runs over HTTPS. `includeSubDomains` applies the policy to all subdomains.

HTTPS and TLS protect the connection. The next section is about what happens when a developer bypasses all of that with a single argument.

---

## `verify=False` — The Most Dangerous Line in Python

```python
# This line has caused more production security incidents than most developers realize
requests.get("https://mybank.com/data", verify=False)
```

This is the single most dangerous pattern in Python network code. It appears in production codebases constantly — added to fix a certificate error and forgotten there permanently. `verify=False` turns off certificate verification entirely. Your application will now connect to any server presenting any certificate — including an attacker sitting between you and the real server, presenting a fraudulent certificate your application accepts without checking. The traffic is still encrypted. But you have agreed to encrypt it with someone whose identity you have never verified.

This is a man-in-the-middle attack. The attacker intercepts your connection, presents their own certificate, forwards your traffic to the real server, and reads every byte in plaintext. Your application never knows anything went wrong.

```
verify=True (correct):
Your app → checks certificate → real mybank.com ✓

verify=False (dangerous):
Your app → skips check → attacker's server
                         Attacker reads everything → forwards to real server
                         Your application sees normal responses ✗
```

### Why developers add it

Every instance of `verify=False` started with one of these:

- Self-signed certificate on an internal service
- Expired certificate that broke a connection
- Local development environment with an untrusted cert
- Corporate proxy presenting its own certificate
- "Just to get it working" — and then it shipped

None of these are reasons to disable verification. They are reasons to fix the underlying certificate problem.

### The correct fix for every scenario

**Self-signed or internal CA:**

```python
# Wrong
requests.get("https://internal-service.company.internal", verify=False)

# Correct — point to the CA bundle
requests.get(
    "https://internal-service.company.internal",
    verify="/Users/youruser/certs/internal-ca.crt"
)
```

**Local development HTTPS:**

Use [mkcert](https://github.com/FiloSottile/mkcert) to generate locally-trusted certificates. It installs a local CA into your system trust store so development certs pass verification without disabling anything.

```bash
# macOS / Linux
brew install mkcert
mkcert -install
mkcert localhost 127.0.0.1

# Windows
choco install mkcert
mkcert -install
mkcert localhost 127.0.0.1
```

**Expired certificate:**

Renew it. An expired certificate is not a configuration problem — it is a maintenance failure. There is no version of this where disabling verification is the right answer. Disabling verification because a certificate lapsed is trading a maintenance problem for a security vulnerability. Fix it at the source.

**Corporate proxy with SSL inspection:**

```python
import os

os.environ['REQUESTS_CA_BUNDLE'] = '/Users/youruser/certs/corporate-ca.crt'

# Or per request
requests.get("https://mybank.com", verify="/Users/youruser/certs/corporate-ca.crt")
```

!!! warning "If you find `verify=False` in a codebase"

    Treat it as a critical finding. Document it. Create a ticket. Fix it before the next release. It is not technical debt — it is an active vulnerability that bypasses the entire trust model TLS was built on.

Your connections are now encrypted and verified. But there is one more failure mode that trips up even experienced developers: what happens when the server on the other end simply stops responding.

---

## Timeouts — Every Request Needs One

Imagine you run a business and you have drivers delivering your product across the city. Every vehicle your application dispatches travels roads it does not control, to buildings it cannot see inside. What happens if the vehicle arrives, knocks on the door, and nobody ever answers?

Without a timeout, your application waits at that door forever.

When your application sends a request, it is trusting that the server will respond within a reasonable amount of time. Servers go down. Networks get congested. And in adversarial scenarios, a malicious actor can deliberately send responses so slowly that your connection stays open indefinitely — consuming resources, blocking threads, and eventually making your application unable to handle anything else.

This is a standard denial-of-service technique. An attacker who controls a server you call can slow-drip responses — sending just enough bytes to keep the connection alive — and hold your processes open until your application falls over.

A timeout is the instruction you give every vehicle before it leaves: if the door is not answered within this amount of time, come back. Do not wait indefinitely.

### What no timeout actually costs you

A thread or process waiting on a network response with no timeout will wait forever. One hung connection is manageable. Under load — or under a deliberate attack — you can accumulate enough hung connections that your application stops being able to process new requests entirely. It becomes unresponsive. It looks like a crash. The actual cause is that every connection is sitting open, waiting on servers that will never answer.

This is a denial-of-service vector. It requires no special exploit — just a slow or unresponsive server, and your missing timeout does the rest.

### Connect timeout vs read timeout

```
Connect timeout — how long to wait for the building to answer the door at all
                  (the initial connection, the TLS handshake)
                  If there is no response to the knock, turn around.

Read timeout    — how long to wait for the building to finish
                  handing over the response after it opened the door
                  If the building opens the door but then goes silent, leave.
```

### How to set timeouts

```python
# Python — requests
# Format: (connect_timeout, read_timeout) in seconds

response = requests.get(
    "https://mybank.com/data",
    timeout=(5, 30)
)
```

```python
# Python — httpx (async)
import httpx

async with httpx.AsyncClient(
    timeout=httpx.Timeout(connect=5.0, read=30.0)
) as client:
    response = await client.get("https://mybank.com/data")
```

```python
# Python — aiohttp (async)
import aiohttp

timeout = aiohttp.ClientTimeout(total=30, connect=5)
async with aiohttp.ClientSession(timeout=timeout) as session:
    async with session.get("https://mybank.com/data") as response:
        data = await response.json()
```

```javascript
// Node — fetch with AbortController
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 10000);

try {
  const response = await fetch("https://mybank.com/data", {
    signal: controller.signal
  });
  const data = await response.json();
} finally {
  clearTimeout(timeoutId);
}
```

```javascript
// Node — axios
const axios = require('axios');

const response = await axios.get("https://mybank.com/data", {
  timeout: 10000  // milliseconds
});
```

### Sensible starting values

```
Connect timeout:   3–5 seconds
Read timeout:      10–30 seconds
Background jobs:   60–120 seconds
```

If an API consistently takes longer than your timeout, the problem is the API — not your timeout. Do not keep raising the limit to accommodate a slow dependency. Fix the integration or raise it with the provider.

---

## The Request Security Checklist

Every network call your application makes should clear this list before it ships to production. Every vehicle that leaves the building, checked.

```
BEFORE ANY OUTBOUND REQUEST SHIPS

→ Does this request use HTTPS?
  If the endpoint only supports HTTP, that is a problem
  with the endpoint — not a reason to use HTTP.
  Do not accept an unencrypted dependency.

→ Is SSL/TLS verification enabled?
  verify=False should not exist anywhere in your codebase.
  If it does, it is a critical finding.

→ Is there a timeout set?
  Both connect and read timeouts, with sensible values.
  An untimed request is a hanging thread waiting to happen —
  and an attacker can hold your vehicles hostage indefinitely.

→ Are credentials in headers — not URLs?
  API keys in URLs appear in server logs, browser history,
  and Referer headers sent to third parties.
  Always use Authorization headers.

→ Is the response validated before use?
  Check status codes. Handle errors explicitly.
  Never assume a 200 response body is what you expected.

→ Is sensitive data being logged from this request?
  Request headers often contain tokens.
  Response bodies often contain user data.
  Log what you need for debugging — not everything.

→ Is this internal endpoint authenticated?
  Internal does not mean trusted.
  Service-to-service calls inside your infrastructure
  should still be authenticated.

→ Are redirects handled safely?
  Some libraries follow redirects automatically.
  Verify that credentials are not being forwarded
  to an unintended destination after a redirect.
```

---

## DNS Security

You know now that every network connection begins with a DNS lookup — your application asking a DNS server to translate `mybank.com` into the IP address it needs to connect. That lookup happens before TLS, before certificates, before any of the protections covered above. It is the first step, and it is a target.

### DNS spoofing — poisoning the address book

DNS spoofing — also called DNS cache poisoning — is when an attacker manipulates the answer your application receives from a DNS query. Instead of returning the real IP address, the poisoned response returns one the attacker controls. Your application connects to the wrong server.

What happens next depends on what protections you have in place:

```
No HTTPS:
Your app connects to attacker's server.
Attacker reads all traffic in plaintext. ✗

HTTPS with verify=False:
Your app connects to attacker's server.
Attacker presents any certificate — your app accepts it.
Attacker reads all traffic in plaintext. ✗

HTTPS with verify=True (correct):
Your app connects to attacker's server.
TLS checks the certificate — name mismatch, not trusted.
Connection is rejected. ✓
```

This is why certificate verification and DNS security are not separate concerns. A correctly configured TLS connection catches a DNS spoofing attack at the certificate check — even if the DNS lookup was poisoned. This is also why `verify=False` is so dangerous: it removes the protection that catches exactly this scenario.

### DNSSEC

DNSSEC (DNS Security Extensions) adds cryptographic signatures to DNS records. When enabled for a domain, resolvers can verify that the DNS response they received was not tampered with in transit.

DNSSEC is configured at the registrar level — it is a setting on your domain, not something you add to application code. Check your domain registrar's documentation for how to enable it. If you own a domain, this is worth doing.

### What your application can do

```python
# Use trusted DNS resolvers explicitly
# Cloudflare (1.1.1.1) and Google (8.8.8.8) support DNS over HTTPS,
# encrypting the query itself so it cannot be read or modified in transit

import dns.resolver  # pip install dnspython

resolver = dns.resolver.Resolver()
resolver.nameservers = ['1.1.1.1', '8.8.8.8']
answer = resolver.resolve('mybank.com', 'A')
```

At the application level, the most reliable defense against DNS attacks is the one already covered: HTTPS with certificate verification enabled. A poisoned DNS response that redirects your traffic does not matter if your TLS check rejects the impersonation.

---

## Data Integrity — Verifying What You Received

Your vehicle made the trip. TLS was in place. The certificate checked out. The building was the right one. You received a response. But is what arrived actually what was sent? TLS protects cargo on the road. Data in transit can be tampered with, corrupted, or substituted — not just read. Once data arrives, you are responsible for checking that it is what it claims to be. Integrity verification is how you confirm that what your application received is what the source actually sent.

### Checksums and hashes for downloaded files

When your application downloads files — release artifacts, configuration data, dependencies — verify the hash of what you received against the expected hash published by the source. If the file was modified anywhere along the path, the hash will not match.

```python
import hashlib
import requests

def download_and_verify(url, expected_sha256, timeout=(5, 30)):
    response = requests.get(url, timeout=timeout)
    response.raise_for_status()

    actual_hash = hashlib.sha256(response.content).hexdigest()

    if actual_hash != expected_sha256:
        raise ValueError(
            f"Hash mismatch — file may have been tampered with.\n"
            f"Expected: {expected_sha256}\n"
            f"Actual:   {actual_hash}"
        )

    return response.content
```

### Webhook signature verification

When an external service pushes data to an endpoint your application exposes — a payment processor, a CI system, a third-party tool — anyone who knows the URL could send data to it. Signature verification is how you confirm a request came from who you expect and has not been modified.

The standard pattern uses HMAC-SHA256: the sender signs the request body with a shared secret and includes the signature in a header. You recompute the expected signature from the raw body and compare. A match confirms authenticity.

```python
import hmac
import hashlib
import os
from flask import Flask, request, abort

app = Flask(__name__)
WEBHOOK_SECRET = os.environ.get("WEBHOOK_SECRET")  # never hardcoded

@app.route("/webhook", methods=["POST"])
def handle_webhook():
    signature_header = request.headers.get("X-Hub-Signature-256", "")

    if not signature_header.startswith("sha256="):
        abort(400, "Missing or malformed signature")

    received_signature = signature_header[7:]

    expected_signature = hmac.new(
        WEBHOOK_SECRET.encode(),
        request.get_data(),  # raw bytes — do not parse the body first
        hashlib.sha256
    ).hexdigest()

    # Constant-time comparison — prevents timing attacks
    if not hmac.compare_digest(received_signature, expected_signature):
        abort(403, "Signature verification failed")

    payload = request.get_json()
    # process payload here
```

!!! warning "Always use `hmac.compare_digest` — not `==`"

    Standard string comparison is vulnerable to timing attacks: an attacker can measure tiny differences in how long the comparison takes to figure out the correct signature one byte at a time. `hmac.compare_digest` takes the same amount of time regardless of where or whether the strings differ.

See [Credential Management →](credentials.md) for guidance on storing webhook secrets securely.

---

## Secure Headers

Every HTTP response your server sends includes headers — instructions to the browser about how to handle your content and what it is allowed to do with it. Think of them like the signs posted at a building's entrance — "no outside food or drinks," "bathrooms for customers only," "no dogs allowed." These are instructions for who and what is permitted inside, and under what conditions. Several of those headers are security controls that reduce your attack surface in meaningful ways. They cost almost nothing to add. They are missing from the majority of applications.

The headers below apply to any web application serving responses to browsers. Set all of them.

```python
# Flask — set all security headers in one place
@app.after_request
def set_security_headers(response):
    # Force HTTPS — never connect over plain HTTP again
    response.headers['Strict-Transport-Security'] = (
        'max-age=31536000; includeSubDomains'
    )
    # Control which resources the browser is allowed to load
    response.headers['Content-Security-Policy'] = (
        "default-src 'self'; "
        "script-src 'self'; "
        "style-src 'self' 'unsafe-inline'; "
        "img-src 'self' data: https:; "
        "connect-src 'self'; "
        "frame-ancestors 'none'"
    )
    # Prevent clickjacking via iframes
    response.headers['X-Frame-Options'] = 'DENY'
    # Prevent browsers from guessing content types
    response.headers['X-Content-Type-Options'] = 'nosniff'
    # Control how much URL info leaks to third parties
    response.headers['Referrer-Policy'] = 'strict-origin-when-cross-origin'
    # Deny browser feature access your app does not need
    response.headers['Permissions-Policy'] = (
        'camera=(), microphone=(), geolocation=()'
    )
    return response
```

```javascript
// Express — helmet handles all of this automatically
const helmet = require('helmet');
app.use(helmet());

// Or set manually
app.use((req, res, next) => {
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  res.setHeader('Content-Security-Policy',
    "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; " +
    "img-src 'self' data: https:; connect-src 'self'; frame-ancestors 'none'"
  );
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  res.setHeader('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');
  next();
});
```

### What each header does

**`Strict-Transport-Security`** — Tells the browser to always connect to your domain over HTTPS, even if someone types `http://` or follows an unencrypted link. The browser enforces this itself for the duration of `max-age`, before any request even leaves the device.

**`Content-Security-Policy`** — Controls exactly which resources — scripts, styles, images, external connections — the browser is allowed to load for your page. A strict CSP is the most effective defense against cross-site scripting (XSS), because even if an attacker injects a script tag into your page, CSP prevents the browser from executing it.

Start with `default-src 'self'` and loosen only what your application actually needs. Use report-only mode to tune the policy before enforcing it:

```python
# Log CSP violations without blocking — use during tuning
response.headers['Content-Security-Policy-Report-Only'] = (
    "default-src 'self'; report-uri /csp-violations"
)
```

**`X-Frame-Options: DENY`** — Prevents your page from being embedded in an `<iframe>` on another site. Without this, an attacker can overlay your page transparently inside their site and trick users into clicking things they did not intend to — a technique called clickjacking.

**`X-Content-Type-Options: nosniff`** — Prevents browsers from guessing the content type of a response. Without it, a browser might decide that a response contains executable code and run it, even if your server delivered it as something else entirely.

**`Referrer-Policy`** — Controls what URL information gets sent in the `Referer` header when users navigate from your site to another. URLs often contain session identifiers, search terms, or internal paths. `strict-origin-when-cross-origin` sends only your domain name — not the full URL — on cross-origin requests.

**`Permissions-Policy`** — Controls which browser APIs your page can access. If your application does not need the camera, microphone, or geolocation, deny them explicitly. If a script is ever injected into your page, this limits what it can do with the user's device.

!!! tip "Verify your headers after deployment"

    Check your live site at [securityheaders.com](https://securityheaders.com). It grades your configuration and explains exactly what is missing and why it matters. Takes thirty seconds and will surface issues you did not know were there.

Headers control what your server tells browsers about how to behave. The last piece is controlling which external origins are allowed to interact with your application from a browser at all — and that is where CORS comes in.

---

## CORS — Cross-Origin Resource Sharing

You have locked down your server's response headers. But there is still a question the browser needs answered before it will let external JavaScript interact with your application: who is actually allowed to make requests here?

Browsers enforce a rule called the same-origin policy: JavaScript running on `app.example.com` is not allowed to make requests to `api.otherdomain.com` unless `api.otherdomain.com` explicitly says it permits that. This rule exists because without it, any website you visit could use your authenticated browser session to make requests to other services you are logged into — reading your data and taking actions on your behalf, invisibly.

CORS is the mechanism that controls those permissions. This is a browser-level protection only — it does not stop direct requests made by servers, curl, scripts, or any non-browser client. But for web applications, it is worth getting right.

### The wildcard `*` problem

```python
# Wrong — allows any website's JavaScript to call your API
response.headers['Access-Control-Allow-Origin'] = '*'
```

`*` means any origin on the internet. Any website can run JavaScript that calls your API in a user's browser, using that user's active session and credentials. For genuinely public APIs with no authentication, `*` is acceptable. For any endpoint that handles credentials, sessions, or user data, it is a misconfiguration.

### Configuring CORS correctly

```python
# Flask — explicit origin allowlist
from flask import Flask, request
from flask_cors import CORS

app = Flask(__name__)
CORS(app, origins=[
    "https://app.example.com",
    "https://staging.example.com"
])

# Or manually for full control
@app.after_request
def set_cors_headers(response):
    allowed_origins = [
        "https://app.example.com",
        "https://staging.example.com"
    ]
    origin = request.headers.get("Origin")
    if origin in allowed_origins:
        response.headers['Access-Control-Allow-Origin'] = origin
        response.headers['Vary'] = 'Origin'
    return response
```

```javascript
// Express — explicit origin allowlist
const cors = require('cors');

const allowedOrigins = [
  'https://app.example.com',
  'https://staging.example.com'
];

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true  // required when sending cookies or Authorization headers
}));
```

!!! tip "The `Vary: Origin` header"

    When `Access-Control-Allow-Origin` is set to a specific origin rather than `*`, add `Vary: Origin` to the response. This tells caches that the response differs by origin — preventing a cached response for one origin from being incorrectly served to a different one.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
