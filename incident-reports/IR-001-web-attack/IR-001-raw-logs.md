# IR-001 — Raw Log Evidence

## Context
These logs were collected from three sources during a security incident
on June 25, 2026 between 02:00 AM and 02:47 AM IST.

**Log Sources:**
- Perimeter Firewall (Palo Alto)
- Web Server (Apache/2.4.49) — DMZ
- Authentication Server — Private Network 1

---

## SOURCE 1: Perimeter Firewall Logs
[2026-06-25 02:00:11] ALLOW | SRC: 185.220.101.45 | DST: 203.0.113.10 |

PROTO: TCP | SRC_PORT: 54821 | DST_PORT: 80 | RULE: INBOUND-WEB
[2026-06-25 02:00:13] ALLOW | SRC: 185.220.101.45 | DST: 203.0.113.10 |

PROTO: TCP | SRC_PORT: 54822 | DST_PORT: 80 | RULE: INBOUND-WEB
[2026-06-25 02:01:45] ALLOW | SRC: 185.220.101.45 | DST: 203.0.113.10 |

PROTO: TCP | SRC_PORT: 54900 | DST_PORT: 80 | RULE: INBOUND-WEB
[2026-06-25 02:03:22] ALERT | SRC: 185.220.101.45 | DST: 203.0.113.10 |

PROTO: TCP | SRC_PORT: 55100 | DST_PORT: 80 | RULE: IDS-HIGH-VOLUME-SCAN
[2026-06-25 02:44:01] ALLOW | SRC: 185.220.101.45 | DST: 203.0.113.10 |

PROTO: TCP | SRC_PORT: 61200 | DST_PORT: 443 | RULE: INBOUND-WEB
[2026-06-25 02:47:33] ALERT | SRC: 10.10.9.5 | DST: 185.220.101.45 |

PROTO: TCP | SRC_PORT: 1433 | DST_PORT: 443 | RULE: PRIVATE-TO-EXTERNAL-BLOCKED

---

## SOURCE 2: Web Server Logs (DMZ — Apache)
185.220.101.45 [2026-06-25 02:00:11] "GET / HTTP/1.1" 200 1245

"Mozilla/5.0 (compatible; Nikto/2.1.6)"
185.220.101.45 [2026-06-25 02:00:13] "GET /admin HTTP/1.1" 404 512

"Mozilla/5.0 (compatible; Nikto/2.1.6)"
185.220.101.45 [2026-06-25 02:00:15] "GET /admin.php HTTP/1.1" 404 512

"Mozilla/5.0 (compatible; Nikto/2.1.6)"
185.220.101.45 [2026-06-25 02:00:18] "GET /wp-admin HTTP/1.1" 404 512

"Mozilla/5.0 (compatible; Nikto/2.1.6)"
185.220.101.45 [2026-06-25 02:00:21] "GET /config.php HTTP/1.1" 200 892

"Mozilla/5.0 (compatible; Nikto/2.1.6)"
185.220.101.45 [2026-06-25 02:00:24] "GET /backup.zip HTTP/1.1" 200 5242880

"Mozilla/5.0 (compatible; Nikto/2.1.6)"
185.220.101.45 [2026-06-25 02:01:45] "HEAD /login.php HTTP/1.1" 200 0

"Mozilla/5.0 (compatible; Nikto/2.1.6)"
185.220.101.45 [2026-06-25 02:03:22] "GET /index.php?page=../../../../etc/passwd HTTP/1.1" 500 1892

"Mozilla/5.0 (compatible; Nikto/2.1.6)"
185.220.101.45 [2026-06-25 02:03:45] "PUT /shell.php HTTP/1.1" 403 0

"Mozilla/5.0 (compatible; Nikto/2.1.6)"
185.220.101.45 [2026-06-25 02:04:10] "DELETE /logs/access.log HTTP/1.1" 403 0

"Mozilla/5.0 (compatible; Nikto/2.1.6)"
185.220.101.45 [2026-06-25 02:15:00] "POST /login.php HTTP/1.1" 401 320

"python-requests/2.26.0"
185.220.101.45 [2026-06-25 02:15:01] "POST /login.php HTTP/1.1" 401 320

"python-requests/2.26.0"
[... 847 identical POST requests returning 401 between 02:15:00 and 02:43:58 ...]
185.220.101.45 [2026-06-25 02:43:59] "POST /login.php HTTP/1.1" 200 4821

"python-requests/2.26.0"
185.220.101.45 [2026-06-25 02:44:01] "GET /dashboard HTTP/1.1" 200 8932

"python-requests/2.26.0"
185.220.101.45 [2026-06-25 02:44:15] "GET /api/users HTTP/1.1" 200 94821

"python-requests/2.26.0"
185.220.101.45 [2026-06-25 02:44:45] "GET /api/transactions HTTP/1.1" 200 524288

"python-requests/2.26.0"

---

## SOURCE 3: Authentication Server Logs (Private Network 1)
[2026-06-25 02:43:59] AUTH_ATTEMPT | USER: admin | SRC_IP: 185.220.101.45 |

STATUS: SUCCESS | SESSION_ID: a7f3k9x2 | PROTO: HTTPS
[2026-06-25 02:44:01] SESSION_ACTIVE | USER: admin | SESSION_ID: a7f3k9x2 |

SRC_IP: 185.220.101.45 | PAGE: /dashboard
[2026-06-25 02:44:15] DATA_ACCESS | USER: admin | SESSION_ID: a7f3k9x2 |

RESOURCE: /api/users | RECORDS_RETURNED: 4821 | SRC_IP: 185.220.101.45
[2026-06-25 02:44:45] DATA_ACCESS | USER: admin | SESSION_ID: a7f3k9x2 |

RESOURCE: /api/transactions | RECORDS_RETURNED: 18432 | SRC_IP: 185.220.101.45
[2026-06-25 02:47:33] OUTBOUND_BLOCKED | SRC: 10.10.9.5 | DST: 185.220.101.45 |

PORT: 443 | RULE: NO-PRIVATE-TO-EXTERNAL | ALERT: DATA_EXFIL_ATTEMPT

