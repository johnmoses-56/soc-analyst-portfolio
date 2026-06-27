# Enterprise Network Architecture — FinanceCorp

## Network Diagram (Text-Based)

```
                            INTERNET
                               │
                               │ (Public IP: 203.0.113.1)
                               ▼
                    ┌─────────────────┐
                    │   ISP ROUTER    │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ PERIMETER       │
                    │ FIREWALL        │  ← Palo Alto Firewall
                    │ (203.0.113.10)  │     Rules: Allow 80, 443 inbound
                    │                 │     Block everything else
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │  LOAD BALANCER  │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │                             │
              ▼                             ▼
    ┌──────────────────┐         ┌──────────────────┐
    │       DMZ        │         │   EMPLOYEE LAN   │
    │  (Public Zone)   │         │  (10.10.2.0/24)  │
    │                  │         │                  │
    │ Web Server       │         │ Workstations     │
    │ (10.10.5.10)     │         │ Internal FW      │
    │ Apache/2.4.49    │         │ (Cisco)          │
    │                  │         └──────────────────┘
    │ NO sensitive     │
    │ data stored here │
    └────────┬─────────┘
             │
    ┌────────▼─────────┐
    │  INTERNAL FW     │
    │  (DMZ → Private) │  ← Cisco Firewall
    └────────┬─────────┘
             │
    ┌────────┴─────────────────────────┐
    │                                  │
    ▼                                  ▼
┌──────────────────┐        ┌──────────────────┐
│  PRIVATE NET 1   │        │  PRIVATE NET 2   │
│  (10.10.8.0/24)  │        │  (10.10.9.0/24)  │
│                  │        │                  │
│ Auth Server      │        │ DB Server        │
│ Login validation │        │ (10.10.9.5)      │
│ Session tokens   │        │ User records     │
│                  │        │ Transactions     │
└──────────────────┘        └──────────────────┘
```

---

## Network Zones Explained

| Zone | IP Range | What Lives Here | Security Level |
|---|---|---|---|
| DMZ | 10.10.5.0/24 | Public web servers — no sensitive data | Low |
| Employee LAN | 10.10.2.0/24 | Staff workstations | Medium |
| Private Net 1 | 10.10.8.0/24 | Authentication servers | High |
| Private Net 2 | 10.10.9.0/24 | Databases — most sensitive | Critical |

---

## Security Devices

| Device | Type | Vendor | Position |
|---|---|---|---|
| Perimeter Firewall | Next-Gen Firewall | Palo Alto | Between internet and DMZ |
| Internal Firewall | Firewall | Cisco | Between DMZ and Private Networks |
| IDS | Intrusion Detection | Snort | Main traffic junction |
| Load Balancer | Traffic Distribution | F5 | In front of web servers |

---

## Traffic Rules (Golden Rules)

| From | To | Should It Happen? | Why |
|---|---|---|---|
| Internet | DMZ (port 80/443) | YES | Normal web traffic |
| Internet | Private Network | NO | Never — critical security violation |
| DMZ | Internet (outbound) | NO | DMZ servers don't initiate connections |
| DMZ | Private Network | NO | Data segregation — critical violation |
| Private Net 2 | Internet | NO | Databases never talk to internet directly |
| Employee LAN | Private Net 1 | YES | Staff need to authenticate |
| Employee LAN | Internet (outbound) | YES (monitored) | Normal browsing |

---

## IP Classification Quick Reference

| IP Range | Type | Class | Used By |
|---|---|---|---|
| 10.0.0.0 – 10.255.255.255 | Private | A | Large enterprises (~16M hosts) |
| 172.16.0.0 – 172.31.255.255 | Private | B | Mid-sized orgs (~1M hosts) |
| 192.168.0.0 – 192.168.255.255 | Private | C | Homes/small offices (~65K hosts) |
| Everything else | Public | — | Internet-routable, globally unique |
| 127.0.0.1 | Loopback | — | Device communicates with itself |
| 255.255.255.255 | Broadcast | — | Send to all devices on local network |
