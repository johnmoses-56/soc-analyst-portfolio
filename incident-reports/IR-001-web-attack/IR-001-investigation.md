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
### Phase 2: Vulnerability Probing (02:03 AM – 02:04 AM)

**What happened:** The attacker attempted two high-risk attack techniques
and one cover-up attempt.

### Key Log Evidence

```
02:03:22 GET /index.php?page=../../../../etc/passwd → 500
02:03:45 PUT /shell.php                             → 403
02:04:10 DELETE /logs/access.log                    → 403
```

---

### GET /index.php?page=../../../../etc/passwd → 500

This is a **Path Traversal Attack**.

The `../../../../` sequence attempts to move outside the web application's
directory and access the Linux system file:

```
/etc/passwd
```

A `500 Internal Server Error` is significant because the server did not simply
reject the request—it encountered an internal error while processing it.

This may indicate:

- The application is vulnerable but another control stopped the exploit.
- The application crashed while handling the malicious input.

Either case requires immediate investigation.

**OSI Layer:** Layer 7 (Application)

---

### PUT /shell.php → 403

The attacker attempted to upload a malicious PHP web shell.

A web shell would allow the attacker to remotely execute commands on the server.

The `403 Forbidden` response confirms the upload was blocked.

Even though the attack failed, PUT requests from external sources are highly
suspicious and should trigger an alert.

---

### DELETE /logs/access.log → 403

The attacker attempted to delete the web server's access logs.

Deleting logs is a common **anti-forensics** technique used to hide evidence
after compromising a system.

The request was blocked (`403 Forbidden`), preserving valuable forensic
evidence for investigators.

This indicates the attacker understood post-exploitation techniques and
attempted to remove traces of their activity.
### Phase 3: Credential Brute Force (02:15 AM – 02:43 AM)

**What happened:** The attacker switched tools and launched an automated
credential stuffing/brute force attack against the login endpoint.

### Tool Change Detected — New User-Agent

```
python-requests/2.26.0
```

The attacker switched from Nikto (scanner) to a custom Python script using
the `requests` library. This is a common brute force setup because it is
lightweight, fast, and easy to automate.

### Key Log Evidence

```
02:15:00  POST /login.php → 401  (failed login #1)
02:15:01  POST /login.php → 401  (failed login #2)

[... 847 failed attempts over 28 minutes ...]

02:43:59  POST /login.php → 200  (SUCCESS)
```

---

### Why POST is Used for Login

HTTP POST sends credentials inside the request body instead of the URL.

This prevents usernames and passwords from appearing in browser history,
proxy logs, bookmarks, and cached URLs.

---

### Understanding the 401 → 200 Pattern

- **401 Unauthorized** = Login failed.
- **847 consecutive 401 responses** = Automated brute force attack.
- **One final 200 response** = Login succeeded.

This pattern is a classic brute-force signature.

The attacker likely obtained valid credentials from the exposed
`backup.zip` file discovered during reconnaissance.

---

### Authentication Server Confirmation

```
02:43:59 AUTH_ATTEMPT
USER: admin
STATUS: SUCCESS
SESSION_ID: a7f3k9x2
```

The Authentication Server confirms that the attacker successfully logged in
as the **admin** user.

A valid session identified by **a7f3k9x2** was created.

From this point onward, the attacker no longer needed to repeatedly send
credentials. Every request carrying this session ID would be treated as an
authenticated administrator.

---

### OSI Layer Significance

This attack primarily affects:

- **Layer 7 (Application)** — Login requests using HTTP POST.
- **Layer 5 (Session)** — A valid authenticated session was established.

Even if the administrator immediately changed the password, the attacker
could remain logged in until the active session was invalidated.

For this reason, incident response must include:

- Session termination
- Password reset
- Account investigation
### Phase 4: Data Access and Exfiltration (02:44 AM – 02:47 AM)

**What happened:** Using the compromised admin session, the attacker
accessed sensitive data endpoints and attempted to exfiltrate the database.

### Key Log Evidence

```
02:44:01  GET /dashboard          → 200 (8,932 bytes)
02:44:15  GET /api/users          → 200 (94,821 bytes)   ⚠️ 4,821 user records
02:44:45  GET /api/transactions   → 200 (524,288 bytes)  ⚠️ 18,432 transaction records
02:47:33  FIREWALL BLOCK: 10.10.9.5 → 185.220.101.45:443 BLOCKED
```

---

### Content-Length Analysis

```
/dashboard           →   8,932 bytes
/api/users           →  94,821 bytes
/api/transactions    → 524,288 bytes
```

The response sizes indicate how much information was returned.

The unusually large responses for `/api/users` and
`/api/transactions` suggest that sensitive information was successfully
retrieved through the compromised administrator account.

---

### Firewall Alert

```
SRC: 10.10.9.5
DST: 185.220.101.45
PORT: 443
RULE: PRIVATE-TO-EXTERNAL-BLOCKED
```

This is the most critical firewall event in the investigation.

The database server (`10.10.9.5`) attempted to establish an outbound HTTPS
connection to the attacker's external IP address.

The firewall correctly blocked the connection because database servers
should never communicate directly with the Internet.

---

### Why This Matters

This event indicates that:

- The attacker successfully authenticated.
- Sensitive records were accessed.
- An outbound data exfiltration attempt occurred.
- Network segmentation prevented a direct database-to-Internet connection.

Although the firewall blocked the outbound connection, sensitive data had
already been accessed through the authenticated web application session.

This confirms a successful compromise requiring immediate incident response.
---

## 5. COMPLETE ATTACK CHAIN

```text
PHASE 1: RECONNAISSANCE (L7 Application)

185.220.101.45 → Nikto scanner → Web Server DMZ
├── Directory enumeration (mass 404s)
├── config.php exposed (200) ← CREDENTIAL LEAK
└── backup.zip downloaded (200, 5.2MB) ← SOURCE CODE/DB LEAK

         ↓

PHASE 2: VULNERABILITY PROBING (L7 Application)

├── Path traversal attempt /etc/passwd (500 — server error, possible vuln)
├── Web shell upload attempt PUT /shell.php (403 — blocked)
└── Log deletion attempt DELETE /logs/access.log (403 — blocked)

         ↓

PHASE 3: BRUTE FORCE (L5 Session + L7 Application)

185.220.101.45 → python-requests → /login.php
├── 847 × POST → 401 (failed attempts)
└── 1 × POST → 200 (SUCCESS — admin credentials compromised)
    └── Session cookie a7f3k9x2 issued

         ↓

PHASE 4: DATA EXFILTRATION (L7 Application + L3 Network)

├── GET /api/users → 200 (4,821 user records sent to attacker)
├── GET /api/transactions → 200 (18,432 financial records sent to attacker)
└── 10.10.9.5 → 185.220.101.45:443 BLOCKED (DB direct exfil attempt — stopped)
```
---

## 6. INDICATORS OF COMPROMISE (IOCs)

| IOC Type | Value | Confidence |
|---|---|---|
| Attacker IP | 185.220.101.45 | High |
| Malicious User-Agent | Nikto/2.1.6 | High |
| Malicious User-Agent | python-requests/2.26.0 | High |
| Compromised Account | admin | Confirmed |
| Compromised Session | a7f3k9x2 | Confirmed |
| Exposed File | /config.php | Confirmed |
| Exposed File | /backup.zip | Confirmed |
| Targeted Endpoints | /api/users, /api/transactions | Confirmed |
| C2 Destination | 185.220.101.45:443 | High |
---

## 7. MITRE ATT&CK MAPPING

| Tactic | Technique | Evidence |
|---|---|---|
| Reconnaissance | T1595 — Active Scanning | Nikto web scanner, mass GET requests |
| Initial Access | T1190 — Exploit Public-Facing Application | Path traversal attempt, config.php access |
| Credential Access | T1110 — Brute Force | 847 failed POSTs → 1 successful |
| Collection | T1213 — Data from Information Repositories | /api/users, /api/transactions accessed |
| Exfiltration | T1041 — Exfiltration Over C2 Channel | DB server → attacker IP:443 (blocked) |
| Defense Evasion | T1070 — Indicator Removal | DELETE /logs/access.log attempt |
---

## 8. IMMEDIATE RESPONSE ACTIONS TAKEN

As SOC Level 1 Analyst, the following immediate actions were taken:

| Priority | Action | Rationale |
|---|---|---|
| 1 | Block 185.220.101.45 at perimeter firewall | Stop ongoing attack |
| 2 | Invalidate session a7f3k9x2 immediately | Terminate attacker's active session |
| 3 | Force password reset on admin account | Compromised credential |
| 4 | Take web server offline for forensic analysis | Preserve evidence |
| 5 | Escalate to SOC L2 and Incident Response team | Confirmed breach |
| 6 | Notify management — data breach possible | 4,821 users + 18,432 transactions |
---

## 9. SPLUNK QUERIES (Conceptual)

The following SPL queries would detect this attack pattern:

```spl
# Detect directory enumeration — mass 404s from single IP
index=web_logs status=404
| stats count by src_ip
| where count > 50
| sort -count

# Detect suspicious user agents
index=web_logs
| search user_agent="*Nikto*" OR user_agent="*sqlmap*" OR user_agent="*python-requests*"
| table _time, src_ip, user_agent, uri, status

# Detect brute force — mass 401s followed by 200
index=web_logs uri="/login.php" status=401
| stats count by src_ip
| where count > 10

# Detect dangerous HTTP methods
index=web_logs method IN ("PUT", "DELETE", "HEAD")
| table _time, src_ip, method, uri, status

# Detect large response bodies (potential exfiltration)
index=web_logs status=200
| where content_length > 50000
| table _time, src_ip, uri, content_length

# Detect private network outbound (critical violation)
index=firewall_logs src_ip="10.10.9.*"
| where NOT match(dest_ip, "^10\.")
| table _time, src_ip, dest_ip, dest_port, action
```
---

## 10. RECOMMENDATIONS

### Immediate (Within 24 Hours)

1. Remove `/config.php` and `/backup.zip` from the web root immediately
2. Audit all files accessible from the web server — remove any sensitive files
3. Implement rate limiting on `/login.php` — block IPs after 5 failed attempts
4. Block User-Agent strings containing `Nikto`, `sqlmap`, `python-requests` at the WAF
5. Suppress Apache version from the Server response header (currently exposing Apache/2.4.49)
6. Patch Apache — version 2.4.49 has critical CVE-2021-41773 path traversal vulnerability

### Short Term (Within 1 Week)

1. Implement Web Application Firewall (WAF) rules for path traversal patterns (`../`)
2. Disable PUT and DELETE methods at the web server configuration level
3. Implement Multi-Factor Authentication (MFA) on all admin accounts
4. Enable session timeout on admin sessions (maximum 30 minutes idle)
5. Move backup files outside the web root entirely — never store them in publicly accessible directories
6. Review and harden network firewall rules — verify Private → External is blocked for all zones

### Long Term (Within 1 Month)

1. Conduct a full penetration test on the web application
2. Implement SIEM alerting for all patterns identified in this report
3. Establish a baseline for normal API response sizes — alert on deviations
4. Implement Privileged Access Management (PAM) for admin account access
5. Review data classification — `/api/users` and `/api/transactions` should require additional authorization beyond a session cookie
---

## 11. LESSONS LEARNED

### What the Attacker Did Well

- Used legitimate-looking user-agent initially
- Switched tools between phases to vary signature
- Targeted high-value endpoints immediately after login
- Attempted evidence destruction (log deletion)
- Used HTTPS (port 443) for C2 to blend with legitimate traffic

### What Our Defenses Did Well

- Perimeter firewall blocked private network outbound traffic
- PUT and DELETE methods were blocked (403)
- Log deletion was blocked (403)
- IDS fired an alert during high-volume scanning phase

### What Failed

- `config.php` was publicly accessible (misconfiguration)
- `backup.zip` was publicly accessible (misconfiguration)
- No rate limiting on the login endpoint (allowed 848 attempts)
- No MFA on the admin account
- Apache version exposed in the Server header
- No WAF rules for common scanning user-agents
