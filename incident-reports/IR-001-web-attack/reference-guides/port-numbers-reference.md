# Port Numbers Reference — SOC Analyst Quick Guide

## Critical Ports Every SOC Analyst Must Know

| Port | Protocol | Service | SOC Notes |
|---|---|---|---|
| 20 | TCP | FTP Data | File transfer — monitor for unusual file transfers |
| 21 | TCP | FTP Control | File transfer control — brute force risk |
| 22 | TCP | SSH | Secure remote login — brute force common attack vector |
| 23 | TCP | Telnet | Unencrypted remote login — should not exist in modern networks |
| 25 | TCP | SMTP | Email sending — phishing relay risk |
| 53 | UDP/TCP | DNS | Domain resolution — DNS tunneling exfiltration risk |
| 80 | TCP | HTTP | Unencrypted web — all traffic visible to interceptors |
| 110 | TCP | POP3 | Email retrieval — unencrypted |
| 143 | TCP | IMAP | Email access — monitor for credential attacks |
| 443 | TCP | HTTPS | Encrypted web — C2 traffic can hide here |
| 445 | TCP | SMB | Windows file sharing — WannaCry/ransomware vector |
| 3306 | TCP | MySQL | Database — should never be exposed externally |
| 3389 | TCP | RDP | Remote Desktop — top attack vector, brute force common |
| 1433 | TCP | MSSQL | Microsoft SQL Server — database exfiltration risk |
| 8080 | TCP | HTTP Alt | Alternative web port — check if legitimate |
| 8443 | TCP | HTTPS Alt | Alternative HTTPS — verify if approved |

---

## Port Ranges

| Range | Name | Description |
|---|---|---|
| 0 – 1023 | Well-Known Ports | Reserved for standard services |
| 1024 – 49151 | Registered Ports | Specific applications |
| 49152 – 65535 | Dynamic/Ephemeral | Client source ports (randomly assigned) |

---

## SOC Alert Triggers by Port

| Scenario | Alert Level |
|---|---|
| External IP → internal port 3389 (RDP) | HIGH |
| Internal machine → external port 3389 | HIGH |
| Any traffic on port 23 (Telnet) | HIGH |
| Database port (1433/3306) exposed to internet | CRITICAL |
| Internal server → external port 443 at regular intervals | HIGH |
| High-volume UDP port 53 with long queries | HIGH |
| Port 445 traffic crossing network segments | HIGH |
