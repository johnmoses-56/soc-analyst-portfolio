# OSI Model Reference — SOC Analyst Quick Guide

## All 7 Layers with SOC-Relevant Fields

| Layer | Name | SOC-Relevant Fields | Key Protocols |
|---|---|---|---|
| 7 | Application | URL, HTTP method, user-agent, hostname | HTTP, HTTPS, FTP, SMTP, DNS |
| 6 | Presentation | Encoding type, TLS version, compression, cipher | TLS/SSL |
| 5 | Session | Cookie, session ID, session duration | Cookies, RPC |
| 4 | Transport | Source port, destination port, TCP flags, TCP/UDP | TCP, UDP |
| 3 | Network | Source IP, destination IP | IP, ICMP |
| 2 | Data Link | MAC address (next hop only) | ARP, Ethernet |
| 1 | Physical | Signal type (cable, Wi-Fi, fiber) | Ethernet, Wi-Fi |

---

## Simplified TCP/IP Model (Real World)

| Layer | Covers OSI | What SOC Analysts Focus On |
|---|---|---|
| Application | L7 + L6 + L5 | HTTP headers, user-agent, cookies, encoding |
| Transport | L4 | TCP/UDP, port numbers, TCP flags |
| Network | L3 | IP addresses, routing |
| Physical | L2 + L1 | MAC addresses (rarely needed in SOC) |

---

## TCP vs UDP — SOC Decision Table

| Protocol | Data Loss | Delay | Use Cases | SOC Relevance |
|---|---|---|---|---|
| TCP | NOT tolerated | Tolerated | Web, email, file transfer, login | Full handshake in logs — SYN flood detection |
| UDP | Tolerated | NOT tolerated | VoIP, DNS, live streaming | Less trace — DNS tunneling risk |

---

## TCP Three-Way Handshake

```
Client → Server: SYN        (I want to connect)
Server → Client: SYN-ACK    (I'm ready)
Client → Server: ACK        (Confirmed — connection established)
```

**SOC Alert:** Mass SYN with no SYN-ACK completion = SYN Flood DDoS attack

---

## Attack Types by OSI Layer

| Attack | Layer | Description |
|---|---|---|
| SYN Flood | L4 Transport | Exhausts server TCP connection table |
| DNS Tunneling | L3/L4 | Data hidden inside DNS queries over UDP port 53 |
| ARP Spoofing | L2 Data Link | Fake MAC associations for MITM attack |
| Session Hijacking | L5 Session | Stolen cookie used to impersonate authenticated user |
| SQL Injection | L7 Application | Malicious SQL in HTTP request parameters |
| Path Traversal | L7 Application | `../` sequences to access restricted files |

---

## Golden Rules for SOC Log Analysis

1. Private → Internet traffic without business justification = investigate
2. DMZ server initiating outbound connections = likely C2 beaconing
3. Private Network → DMZ traffic = critical red flag
4. Mass SYN with no ACK = SYN flood
5. High-volume UDP port 53 with long subdomains = DNS tunneling
6. Regular beaconing intervals to unknown IP = C2 malware
