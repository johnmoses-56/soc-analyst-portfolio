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
