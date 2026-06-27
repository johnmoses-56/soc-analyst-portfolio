# Incident Report IR-001
## Web Application Attack — FinanceCorp

---

| Field | Details |
|---|---|
| **Report ID** | IR-001 |
| **Analyst** | John Moses (SOC Level 1) |
| **Date of Incident** | June 25, 2026 |
| **Time of Incident** | 02:00 AM – 02:47 AM IST |
| **Severity** | CRITICAL |
| **Status** | Confirmed Breach — Escalated to L2 |
| **Systems Affected** | Web Server (DMZ), Authentication Server (Private Net 1), Database Server (Private Net 2) |

---

## 1. EXECUTIVE SUMMARY

On June 25, 2026 between 02:00 AM and 02:47 AM IST, FinanceCorp's web
application was targeted in a multi-stage attack. The attacker, originating
from external IP `185.220.101.45`, conducted web reconnaissance using an
automated scanning tool, followed by a credential brute force attack against
the login portal.

The attack succeeded at 02:43:59 AM when the attacker gained unauthorized
access to the `admin` account. Following this, the attacker accessed sensitive
user and financial transaction records. A subsequent attempt to exfiltrate data
directly from the database server was blocked by firewall rules.

**Total records potentially exposed:** 4,821 user records + 18,432 transaction records.

---

## 2. NETWORK CONTEXT

Before analyzing the logs, it is essential to understand where each system
sits in the network architecture.

### Network Zone Classification

```
INTERNET (185.220.101.45 — ATTACKER)
       │
       ▼
PERIMETER FIREWALL (Palo Alto)
       │
       ▼
WEB SERVER — 10.10.5.10 — DMZ Zone (Public-facing, low sensitivity)
       │
       ▼
INTERNAL FIREWALL (Cisco)
       │
       ├──► AUTH SERVER — 10.10.8.x — Private Network 1 (Login validation)
       │
       └──► DB SERVER — 10.10.9.5 — Private Network 2 (Sensitive data — CRITICAL)
```

### Why Network Zone Matters for This Investigation

The DMZ (Demilitarized Zone) is designed to host public-facing web servers
with NO sensitive data. It exists so that if a web server is compromised,
the attacker is isolated from internal systems.

In this incident, the attacker successfully moved beyond the DMZ into the
Private Network — which means the attack bypassed the primary purpose of
network segmentation. This is why severity is rated CRITICAL.

**Key network security rules violated in this incident:**

- Private Network data (10.10.9.5) attempted to flow to external internet — BLOCKED
- Attacker gained access to Private Network 1 (authentication zone) from internet
- Database records (Private Network 2) were accessed by an unauthenticated external party
---

## 3. OSI MODEL MAPPING

Understanding which OSI layer each attack phase occurred at helps us
understand what tools and controls would detect or prevent each phase.

| Attack Phase | OSI Layer | Why This Layer |
|---|---|---|
| Port scanning / reconnaissance | L4 Transport | Attacker probing destination ports |
| HTTP request scanning | L7 Application | GET/HEAD requests to web server |
| User-agent fingerprinting | L6 Presentation | Encoding/tool identification in headers |
| Brute force login | L5 Session | Cookie/session token being targeted |
| Credential validation | L7 Application | POST request to login endpoint |
| Data access via API | L7 Application | GET requests to /api endpoints |
| Data exfiltration attempt | L3 Network | Private IP attempting outbound connection |

---

## 4. ATTACK TIMELINE

### Phase 1: Reconnaissance (02:00 AM – 02:04 AM)

**What happened:** The attacker used Nikto — an automated web vulnerability
scanner — to map the web application, identify valid pages, and probe for
known vulnerabilities.

### How we know it's Nikto

The User-Agent string in every request reads:

```
Mozilla/5.0 (compatible; Nikto/2.1.6)
```

Legitimate users use browsers (Chrome, Firefox, Safari).
The string `Nikto/2.1.6` identifies this as an automated scanning tool,
not a human browser. User-agent strings can be spoofed, but in this case
the request pattern (hundreds of requests in seconds to common vulnerability
paths) confirms automated scanning.

### Key Log Evidence

```
02:00:11 GET /              → 200  (homepage found — valid target confirmed)
02:00:13 GET /admin         → 404  (not found — attacker mapping structure)
02:00:15 GET /admin.php     → 404  (not found)
02:00:18 GET /wp-admin      → 404  (not found — tested for WordPress)
02:00:21 GET /config.php    → 200  ⚠️ CONFIG FILE EXPOSED
02:00:24 GET /backup.zip    → 200  ⚠️ BACKUP FILE DOWNLOADED (5.2 MB)
02:01:45 HEAD /login.php    → 200  (confirmed login page exists)
```

### Analysis of Each Request

**GET / → 200**

The attacker confirmed the server was online and responding.
HTTP GET retrieves a web page. A `200 OK` response means the homepage
was successfully served, confirming the target was reachable.

---

**GET /admin → 404**

**GET /admin.php → 404**

**GET /wp-admin → 404**

These are classic **directory enumeration** attempts.

The attacker systematically requested common administrative paths to
discover hidden pages.

A `404 Not Found` response means the requested page does not exist,
but repeated 404s from one IP are a common reconnaissance indicator.

---

**GET /config.php → 200**

🚨 **Critical Finding**

A configuration file was successfully downloaded.

Configuration files often contain:

- Database credentials
- API keys
- Internal IP addresses
- Application secrets

These files should never be publicly accessible.

---

**GET /backup.zip → 200**

🚨 **Critical Finding**

A **5.2 MB backup archive** was downloaded.

Large backup files commonly contain:

- Source code
- Database dumps
- Configuration files
- Credentials

This single event may have provided everything needed for later attack stages.

---

**HEAD /login.php → 200**

Browsers rarely send `HEAD` requests.

Security scanners frequently use HEAD requests because they retrieve only
the HTTP headers without downloading the page content.

This confirmed that the login page existed while minimizing traffic.

This marks the transition from passive reconnaissance to active targeting.
