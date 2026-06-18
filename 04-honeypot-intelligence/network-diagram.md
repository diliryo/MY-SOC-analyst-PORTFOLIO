# Honeypot Network Architecture

## Network Topology

```
╔═══════════════════════════════════════════════════════════════════════╗
║                          INTERNET                                      ║
╚═════════════════════════════╤═════════════════════════════════════════╝
                              │
                    ┌─────────┴─────────┐
                    │   Internet Edge   │
                    │  (ISP Router)     │
                    └─────────┬─────────┘
                              │
        ┌─────────────────────┴──────────────────────┐
        │                                            │
   ┌────▼──────┐                           ┌────────▼────┐
   │ Firewall  │                           │   Switch    │
   │ (pfSense) │                           │  (Managed)  │
   └────┬──────┘                           └─────────────┘
        │
        │ Port Forward: 22, 80, 443
        │
   ┌────▼──────────────────────────────────────────┐
   │         DMZ Network (VLAN 10)                  │
   │      10.99.99.0/24 (Isolated)                 │
   ├──────────────────────────────────────────────┤
   │                                               │
   │  ┌──────────────┐  ┌──────────────┐          │
   │  │  Honeypot 1  │  │  Honeypot 2  │          │
   │  │  SSH Server  │  │  HTTP Server │          │
   │  │ 10.99.99.10  │  │ 10.99.99.11  │          │
   │  │              │  │              │          │
   │  │ Services:    │  │ Services:    │          │
   │  │ - OpenSSH    │  │ - Apache     │          │
   │  │ - Syslog Clnt│  │ - Syslog Clnt│          │
   │  │ - Suricata   │  │ - Suricata   │          │
   │  └──────────────┘  └──────────────┘          │
   │                                               │
   │  ┌──────────────┐  ┌──────────────┐          │
   │  │  Honeypot 3  │  │  Log Server  │          │
   │  │  DNS Server  │  │  (Syslog)    │          │
   │  │ 10.99.99.12  │  │ 10.99.99.100 │          │
   │  │              │  │              │          │
   │  │ Services:    │  │ Services:    │          │
   │  │ - BIND       │  │ - Rsyslog    │          │
   │  │ - Syslog Clnt│  │ - tcpdump    │          │
   │  │ - Suricata   │  │ - Logwatch   │          │
   │  └──────────────┘  └──────────────┘          │
   │                                               │
   └──────────────────────────────────────────────┘
              │
              │ Access Control: BLOCKED outbound
              │ (DMZ ↔ Production: Firewall rule denies)
              │
   ┌──────────▼──────────────────────────────────┐
   │    Production Network (VLAN 20)             │
   │      192.168.1.0/24 (Main Office)           │
   ├────────────────────────────────────────────┤
   │                                             │
   │  Production systems (no access from DMZ)   │
   │  - Domain Controller                       │
   │  - File Server                             │
   │  - Workstations                            │
   │                                             │
   └─────────────────────────────────────────────┘
```

## Network Segmentation Rules

### Firewall Rules (pfSense)

| Direction | Source | Destination | Protocol | Port | Action | Notes |
|---|---|---|---|---|---|---|
| Inbound | Internet | 10.99.99.10 | TCP | 22 | Allow | SSH honeypot |
| Inbound | Internet | 10.99.99.11 | TCP | 80, 443 | Allow | HTTP(S) honeypot |
| Inbound | Internet | 10.99.99.12 | UDP | 53 | Allow | DNS honeypot |
| Outbound | 10.99.99.0/24 | 10.99.99.100 | TCP | 514 | Allow | Syslog logging |
| Outbound | 10.99.99.0/24 | 192.168.1.0/24 | All | All | **DENY** | **Critical: DMZ cannot reach production** |
| Outbound | 10.99.99.0/24 | Internet | TCP | 443 | Allow | Threat intel feeds (controlled) |

## Device Specifications

### Honeypot 1: SSH Server
- **OS:** Linux (Debian 11)
- **RAM:** 2GB
- **Disk:** 50GB
- **Network:** 10.99.99.10/24
- **Default GW:** 10.99.99.1
- **DNS:** 10.99.99.12
- **Logging:** Syslog to 10.99.99.100:514

### Honeypot 2: HTTP Server
- **OS:** Linux (Ubuntu 20.04 LTS)
- **RAM:** 2GB
- **Disk:** 100GB (web content)
- **Network:** 10.99.99.11/24
- **Default GW:** 10.99.99.1
- **DNS:** 10.99.99.12
- **Logging:** Syslog to 10.99.99.100:514

### Honeypot 3: DNS Server
- **OS:** Linux (CentOS 8)
- **RAM:** 1GB
- **Disk:** 50GB
- **Network:** 10.99.99.12/24
- **Default GW:** 10.99.99.1
- **Primary DNS:** 8.8.8.8 (for upstream queries)
- **Logging:** Syslog to 10.99.99.100:514

### Log Server
- **OS:** Linux (Ubuntu 20.04 LTS)
- **RAM:** 4GB
- **Disk:** 500GB (log storage)
- **Network:** 10.99.99.100/24
- **Default GW:** 10.99.99.1
- **NTP Server:** pool.ntp.org (time sync critical for correlation)

## Data Flow

### Attack Traffic Flow
```
Internet Attacker
    ↓
[ISP Router] → [Port Forwarding]
    ↓
[Firewall] → [Allowed to honeypot IPs]
    ↓
[Honeypot] → [SSH/HTTP/DNS services]
    ↓
[Activity Logged] → [Syslog sent to 10.99.99.100:514]
    ↓
[Log Server] → [Files: /var/log/honeypot/]
```

### Evidence Collection

**Real-time:**
- Syslog streams to log server
- Suricata IDS alerts to SIEM
- tcpdump packet captures

**Post-Analysis:**
- SSH auth logs: `/var/log/auth.log`
- HTTP access logs: `/var/log/apache2/access.log`
- DNS queries: `/var/log/named/queries.log`
- System logs: `/var/log/syslog`
- Full packet captures: `/var/log/pcaps/` (tcpdump)

## Critical Design Decisions

1. **DMZ Isolation** — Honeypots cannot reach production network; prevents attacker pivot
2. **Centralized Logging** — All honeypot logs sent to isolated log server for analysis
3. **No Outbound Internet** — Except controlled channels (prevents attacker C2 to real servers)
4. **VLANs** — Segregation at switch level for additional isolation
5. **Packet Capture** — Full PCAP retained for forensic analysis
6. **Time Sync** — NTP ensures log correlation accuracy across systems

---

*Network diagram represents lab deployment. Production deployments should add:
- Redundant log storage
- Multiple honeypots in different geographic locations
- Automated threat intel integration
- Real-time alerting to SOC
*