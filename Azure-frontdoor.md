Azure Front Door is an advanced content delivery network (CDN) for the cloud. It's designed to provide fast, reliable, and secure access to your applications' static and dynamic web content globally. By using Microsoft's extensive global edge network, Azure Front Door provides efficient content delivery through global and local points of presence (PoPs) strategically positioned close to both enterprise and consumer users.

What is Azure Front Door?
Azure Front Door (AFD) is a global, cloud-native application delivery network (ADN) service from Microsoft Azure. Think of it as a smart, global traffic cop sitting in front of your web applications — it receives all incoming user requests and intelligently routes them to the best available backend, all while providing security, caching, and load balancing.
It operates at Layer 7 (HTTP/HTTPS) of the OSI model, meaning it understands web traffic deeply.

Why do we use it?
Imagine you've deployed your web app in multiple Azure regions (say, East US, West Europe, Southeast Asia). Without Front Door, users in India hit the US server — slow. With Front Door, they get routed to the nearest healthy backend automatically. Here's what it solves:
Performance — Routes users to the nearest or fastest backend using Microsoft's global edge network (150+ points of presence). Latency drops dramatically.
High Availability — If one region goes down, traffic is automatically failed over to a healthy region. Zero manual intervention.
Security — Built-in Web Application Firewall (WAF), DDoS protection, SSL termination, and bot protection.
Scalability — Handles massive traffic spikes globally. CDN caching reduces load on origin servers.
Simplified management — One entry point for a globally distributed app. No need to manage separate load balancers per region.

C:\Users\307388\Downloads\azure_front_door_flow.svg

# Azure Front Door — Complete Documentation

## Table of Contents

- [What is Azure Front Door?](#what-is-azure-front-door)
- [Why Use Azure Front Door?](#why-use-azure-front-door)
- [How It Works — Request Lifecycle](#how-it-works--request-lifecycle)
  - [Stage 1 — Anycast DNS & Edge PoP Selection](#stage-1--anycast-dns--edge-pop-selection)
  - [Stage 2 — SSL Termination at the Edge](#stage-2--ssl-termination-at-the-edge)
  - [Stage 3 — WAF Inspection](#stage-3--waf-inspection)
  - [Stage 4 — Edge Cache Lookup](#stage-4--edge-cache-lookup)
  - [Stage 5 — Route Matching & URL Rewrite](#stage-5--route-matching--url-rewrite)
  - [Stage 6 — Origin Health Probe Check](#stage-6--origin-health-probe-check)
  - [Stage 7 — Backend Selection & Forwarding](#stage-7--backend-selection--forwarding)
  - [Stage 8 — Response Path & Caching](#stage-8--response-path--caching)
- [Key Features](#key-features)
- [Real-World Example](#real-world-example)
- [Inside Azure Front Door — What Each Layer Does](#inside-azure-front-door--what-each-layer-does)
- [Alternate / Competing Options](#alternate--competing-options)
- [When to Use AFD vs Alternatives](#when-to-use-afd-vs-alternatives)

---

## What is Azure Front Door?

Azure Front Door (AFD) is a **global, cloud-native application delivery network (ADN)** service from Microsoft Azure. It acts as a smart, global entry point sitting in front of your web applications — receiving all incoming user requests and intelligently routing them to the best available backend, while providing security, caching, and load balancing.

It operates at **Layer 7 (HTTP/HTTPS)** of the OSI model, meaning it understands web traffic deeply and can make routing decisions based on URLs, headers, query strings, and geography.

---

## Why Use Azure Front Door?

| Problem | How AFD Solves It |
|---|---|
| Users in India hitting a US server (high latency) | Routes users to the nearest healthy backend via edge PoPs |
| Backend goes down in one region | Automatic failover to another region within seconds |
| App under DDoS or bot attack | Built-in WAF, DDoS protection, bot scoring at the edge |
| High load on origin servers | CDN caching reduces origin calls by 70–90% |
| Managing SSL certs across regions | Centralized cert management with auto-renewal |
| Multiple backends hard to manage | Single entry point with one domain for global apps |

---

## How It Works — Request Lifecycle

When a user makes a request (e.g. `https://myapp.com/api/products`), it passes through 8 internal stages inside Azure Front Door before a response is returned.

```
User Request
     │
     ▼
[1] Anycast DNS → Nearest Edge PoP
     │
     ▼
[2] TLS / SSL Termination
     │
     ▼
[3] WAF Inspection (block malicious traffic)
     │
     ▼
[4] Edge Cache Lookup → Cache HIT? ──────────────────────► Return cached response
     │ Cache MISS                                              to user
     ▼
[5] Route Matching & URL Rewrite
     │
     ▼
[6] Origin Health Probe Check
     │
     ▼
[7] Load Balancing → Select healthy backend
     │
     ▼
[8] Forward over Microsoft Private WAN → Backend Origin
     │
     ▼
Response flows back → Edge caches it → User receives response
```

---

### Stage 1 — Anycast DNS & Edge PoP Selection

When a user's browser resolves your domain, DNS returns Azure Front Door's **Anycast IP**. All 150+ edge PoPs share this same IP address. The network automatically routes the user's packets to the **geographically closest PoP** — not your backend server. The TCP connection is terminated right there at the edge, dramatically cutting round-trip latency.

| Property | Detail |
|---|---|
| Mechanism | Anycast BGP routing |
| Edge locations | 150+ global PoPs |
| Protocol | TCP terminated at edge |
| Key benefit | Nearest PoP = lower RTT for users |

---

### Stage 2 — SSL Termination at the Edge

TLS encryption is **offloaded at the edge PoP**, not at your backend. AFD manages your SSL certificates (including auto-renewal via Azure-managed certs), handles the TLS handshake, and decrypts HTTPS traffic. The onward connection to your backend can be re-encrypted with a fresh TLS session, or travel over Microsoft's private WAN (already secure).

| Property | Detail |
|---|---|
| Cert management | Azure-managed or BYO custom cert |
| Supported protocols | TLS 1.2 / TLS 1.3 |
| Backend transport | Private WAN or HTTPS re-encryption |
| Key benefit | No SSL overhead on origin servers |

---

### Stage 3 — WAF Inspection

Before the request goes anywhere, the **Web Application Firewall** inspects every byte. AFD's WAF runs Microsoft's managed rule sets (covering OWASP Top 10: SQL injection, XSS, path traversal, etc.) plus your own custom rules. Bot protection scores each request by behaviour patterns. Malicious requests are blocked or rate-limited at the edge — before they ever touch your backend.

| Property | Detail |
|---|---|
| Managed rule sets | OWASP 3.2 + Microsoft default |
| Bot protection | Behaviour-based scoring |
| DDoS protection | Layer 7 rate limiting |
| Actions available | Allow / Block / Log / Redirect |

**Common WAF protections:**

- SQL injection (`' OR 1=1 --`)
- Cross-site scripting (XSS)
- Path traversal (`../../etc/passwd`)
- Remote file inclusion
- Bad bot filtering
- IP reputation blocking

---

### Stage 4 — Edge Cache Lookup

AFD checks its **edge cache**. If the response for this request (URL + headers) was cached and hasn't expired, it returns it instantly — without touching your backend at all.

| Property | Detail |
|---|---|
| Cache key | URL + query string + headers |
| TTL control | Per-route rules (seconds to days) |
| Cache bypass | Auth'd requests, POST methods |
| Typical hit rate | 70–90% for content-heavy apps |

**Cache rule examples:**

```
/static/*        → Cache for 7 days
/api/products    → Cache for 60 seconds
/api/user/*      → No cache (authenticated)
```

---

### Stage 5 — Route Matching & URL Rewrite

If there's a cache miss, AFD matches the request path against your **routing rules**. Rules are evaluated top-down by priority.

| Property | Detail |
|---|---|
| Match criteria | Path, HTTP header, query string, geography |
| Actions | Route to origin, rewrite URL, redirect |
| Evaluation order | Priority-based, top-down |

**Example routing rules:**

| Pattern | Action | Destination |
|---|---|---|
| `/api/*` | Forward | East US App Service |
| `/images/*` | Forward | SE Asia CDN origin |
| `/` | Forward | Primary backend |
| `http://*` | Redirect | `https://` (301) |
| `/v1/*` | Rewrite URL | Strip `/v1` prefix, forward |

---

### Stage 6 — Origin Health Probe Check

AFD continuously probes all backends every 30 seconds (configurable). If a backend fails the health threshold, AFD **automatically marks it unhealthy** and stops sending traffic — failover happens within seconds, with no manual intervention.

| Property | Detail |
|---|---|
| Probe type | HTTP HEAD or GET |
| Default interval | Every 30 seconds |
| Failure threshold | Configurable (e.g. 3 consecutive failures) |
| Failover time | Seconds (automatic) |
| Latency scoring | Used to rank backends for routing |

---

### Stage 7 — Backend Selection & Forwarding

Among healthy backends in an origin group, AFD picks one based on your **load balancing method**. The request is then forwarded over Microsoft's **private global WAN** — not the public internet.

| Property | Detail |
|---|---|
| Load balancing methods | Weighted round-robin / Latency-based / Session affinity |
| Transport | Microsoft private WAN |
| Supported origin types | App Service, AKS, VM, Public IP, Storage |
| Private Link support | Backend needs no public IP |

**Load balancing methods explained:**

- **Weighted** — Split traffic by percentage (e.g. 80% East US, 20% West Europe for canary deploys)
- **Latency-based** — Always send to the fastest-responding backend (measured by health probes)
- **Session affinity** — Sticky sessions; same user always hits the same backend

---

### Stage 8 — Response Path & Caching

The backend responds to AFD. AFD checks response headers — if the origin sends `Cache-Control: max-age=3600`, AFD stores the response at the edge PoP. The response travels back to the user over the existing TLS connection.

| Property | Detail |
|---|---|
| Cache write trigger | `Cache-Control` response header |
| Compression | Gzip / Brotli applied at edge |
| Typical global latency | 50–150ms end-to-end |
| Connection reuse | Persistent keep-alive |

---

## Key Features

### Anycast Routing
All PoPs share the same IP. DNS and BGP automatically pick the closest edge node for each user — no geo-DNS configuration needed.

### Origin Groups
You group your backends (origins) and define health probe thresholds, traffic weights, and failover priority. One origin group can contain backends across multiple Azure regions.

### Routing Rules
Path-based, header-based, query string, and geography-based routing. Each rule maps a match condition to an action (forward, rewrite, redirect).

### URL Rewrite & Redirect
AFD can strip path prefixes, redirect HTTP to HTTPS, and rewrite query strings before forwarding — without any code changes on your backend.

### Private Link Origins
Your backend does not need a public IP. AFD connects via Azure Private Link, keeping origin traffic entirely within the Azure network.

### Web Application Firewall (WAF)
Managed OWASP rule sets, custom rules, bot protection, IP allow/block lists, and rate limiting — all configurable per-route.

### CDN Caching
Per-route cache rules with configurable TTL, query string handling, and on-demand cache purge via the Azure Portal or REST API.

---

## Real-World Example

**Scenario:** E-commerce app deployed in 3 Azure regions.

```
                    ┌─────────────────────┐
    Users ──────────►  Azure Front Door   │
    (global)        │  myshop.com         │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
        SE Asia             East US         West Europe
        App Service         App Service     App Service
        (primary for IN)    (for US users)  (for EU users)
```

**What AFD handles automatically:**

- Indian users → routed to SE Asia backend (lowest latency)
- US users → routed to East US backend
- SE Asia backend goes down at 2 AM → AFD detects failure within 30s, silently shifts Indian users to East US
- `/images/*` requests → served from edge cache, origin never called
- WAF blocks a SQL injection attempt before it reaches any backend
- SSL cert expires → AFD auto-renews, zero downtime

**Configuration summary:**

```yaml
Frontend:
  hostname: myshop.com
  certificate: azure-managed

Routes:
  - pattern: /images/*
    cache: 7 days
    origin-group: global-app

  - pattern: /api/*
    cache: none
    origin-group: global-app

  - pattern: /*
    cache: 60 seconds
    origin-group: global-app

Origin Group (global-app):
  load-balancing: latency-based
  health-probe: HTTP HEAD /health every 30s
  origins:
    - SE Asia App Service  (weight: 40)
    - East US App Service  (weight: 40)
    - West Europe App Service (weight: 20)
```

---

## Inside Azure Front Door — What Each Layer Does

| Layer | Component | What it does | Where it runs |
|---|---|---|---|
| 1 | Anycast DNS | Routes user to nearest PoP | Global DNS / BGP |
| 2 | TLS termination | Handles SSL handshake & certs | Edge PoP |
| 3 | WAF | Blocks attacks & bad bots | Edge PoP |
| 4 | Cache | Returns cached responses | Edge PoP |
| 5 | Routing engine | Matches rules, rewrites URLs | Edge PoP |
| 6 | Health probes | Monitors backend availability | AFD control plane |
| 7 | Load balancer | Picks the best healthy backend | Edge PoP |
| 8 | Private WAN | Forwards request to origin | Microsoft backbone |

---

## Alternate / Competing Options

| Service | Provider | Best for | Key difference vs AFD |
|---|---|---|---|
| **Azure Application Gateway** | Azure | Single-region L7 LB | No global routing; lives inside a VNet |
| **Azure Traffic Manager** | Azure | DNS-level global routing | Works at DNS only, no WAF or CDN caching |
| **Azure CDN** (Akamai/Verizon) | Azure | Static content delivery | No routing logic or WAF |
| **AWS CloudFront** | AWS | AWS workloads | Amazon's AFD equivalent |
| **AWS Global Accelerator** | AWS | Low-latency TCP/UDP routing | Layer 4 (not HTTP-aware), no WAF |
| **Cloudflare** | Third-party | Multi-cloud WAF + CDN | Not Azure-native; works across any cloud |
| **Google Cloud LB** | GCP | GCP workloads | GCP's equivalent |
| **NGINX / HAProxy** | Self-managed | Full control needed | You manage infrastructure |

---

## When to Use AFD vs Alternatives

**Use Azure Front Door when:**
- Your app is globally distributed across multiple Azure regions
- You need HTTP-level routing, WAF, and CDN caching in a single managed service
- You want automatic zero-config failover across regions
- You need path-based or geo-based routing with SSL offload

**Use Application Gateway when:**
- Your app is in a single Azure region
- You need L7 routing + WAF within a VNet
- You don't need global traffic distribution

**Use Traffic Manager when:**
- You just need DNS-level geo-routing
- Your backends are not all Azure services
- You don't need HTTP inspection, caching, or WAF

**Use Cloudflare when:**
- You have a multi-cloud setup (Azure + AWS + on-prem)
- You want a vendor-agnostic WAF and CDN
- You need advanced bot management beyond what AFD provides

---

*Last updated: May 2026*