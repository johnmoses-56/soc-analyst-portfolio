# HTTP Response Codes — SOC Analyst Reference

## The Five Families

| Range | Category | SOC Action |
|---|---|---|
| 2XX | Success | Baseline normal — monitor for unusual content-length |
| 3XX | Redirection | Check redirect destination — open redirect risk |
| 4XX | Client Error | Investigate patterns — scanning/brute force indicators |
| 5XX | Server Error | Immediate alert — business impact + possible exploit |

---

## Critical Codes to Memorize

### 2XX — Success

| Code | Name | SOC Significance |
|---|---|---|
| 200 | OK | Normal — but large content-length to external IP = exfiltration risk |

### 3XX — Redirection

| Code | Name | SOC Significance |
|---|---|---|
| 301 | Moved Permanently | HTTP → HTTPS redirect (normal). Check destination |
| 302 | Found (Temporary) | Monitor redirect chains — open redirect abuse risk |

### 4XX — Client Errors

| Code | Name | SOC Significance |
|---|---|---|
| 400 | Bad Request | Malformed headers — possible fuzzing/tampering |
| 401 | Unauthorized | Not authenticated. Mass 401s = brute force in progress |
| 403 | Forbidden | Authenticated but blocked. Internal 403s = privilege escalation attempt |
| 404 | Not Found | Mass 404s from one IP = directory enumeration / scanning |

### 5XX — Server Errors

| Code | Name | SOC Significance |
|---|---|---|
| 500 | Internal Server Error | Possible exploit trigger — SQL injection, path traversal |
| 502 | Bad Gateway | Infrastructure issue or upstream attack |
| 503 | Service Unavailable | Server overloaded — possible DDoS in progress |

---

## Attack Patterns by Response Code

| Attack Type | Pattern in Logs |
|---|---|
| Directory Enumeration | Mass 404s from single IP in short time |
| Brute Force | Mass 401s → then 200 on POST to login |
| SQL Injection | 500 errors following requests with SQL syntax in URL |
| DDoS | Spike in 503 responses across all users |
| Successful Breach | 401 × many → 200 × 1 on POST login |
| Data Exfiltration | 200 responses with abnormally large content-length |

---

## HTTP Methods — SOC Risk Classification

| Method | Risk Level | SOC Action |
|---|---|---|
| GET | Normal | Baseline — monitor volume and URL patterns |
| POST | Normal | Monitor login endpoints for brute force |
| HEAD | Suspicious | Investigate — tools/scanners use HEAD, not browsers |
| PUT | Dangerous | Alert immediately — file upload attempt |
| DELETE | Dangerous | Alert immediately — data destruction attempt |
