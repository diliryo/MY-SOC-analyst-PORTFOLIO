# Wireshark Filter Cookbook

**Quick reference for common Wireshark display filters used in security analysis.**

## Traffic Filtering

### By IP Address
```wireshark
# All traffic to/from specific IP
ip.addr == 192.168.1.10

# All traffic from specific source
ip.src == 45.77.65.211

# All traffic to specific destination
ip.dst == 8.8.8.8

# Traffic between two IPs
ip.src == 192.168.1.10 && ip.dst == 192.168.1.5
```

### By Port
```wireshark
# Specific port (any direction)
tcp.port == 445

# Port range
tcp.port >= 3000 && tcp.port <= 5000

# Multiple ports (SMB lateral movement)
tcp.port == 139 || tcp.port == 445

# High ports (suspicious outbound)
tcp.dstport > 5000 && tcp.dstport < 65535
```

## Protocol-Specific Filters

### SMB (Port 445) — Lateral Movement Detection
```wireshark
# All SMB traffic
smb

# SMB authentication attempts
smb.cmd == 0x73  # Session Setup AndX

# Named pipe access (lateral movement indicator)
smb.pipe

# SMB with specific credentials
smb.acct contains "svc_account"
```

### DNS (Port 53) — Tunneling & C2 Detection
```wireshark
# All DNS queries
dns

# Unusual query types (TXT, SRV indicate tunneling)
dns.qtype == 16  # TXT records
dns.qtype == 33  # SRV records

# DNS queries with suspicious patterns
dns.qry.name contains "exfil"
dns.qry.name contains "c2"

# High-frequency queries from single source
dns && ip.src == 192.168.1.10
```

### HTTP (Port 80/443) — Web-based Attacks
```wireshark
# All HTTP traffic (unencrypted)
http

# Specific HTTP methods
http.request.method == "POST"  # Likely C2 callback or exfil
http.request.method == "PUT"   # File upload

# Large responses (possible data exfil)
http.response.body.size > 5000000

# Specific User-Agent (malware fingerprint)
http.user_agent contains "Mozilla/5.0 (Windows NT 10.0; rv:68.0)"
```

### HTTPS (Port 443) — Encrypted C2
```wireshark
# All HTTPS/TLS traffic
ssl || tls

# Suspicious certificates
tls.handshake.certificate contains "CN=internal"

# Long-lived connections (C2 beaconing)
frame.time_delta > 60  # Packets >60s apart
```

## Anomaly Detection Filters

### Unusual Packet Sizes
```wireshark
# Large packets (potential exfil)
frame.len > 1000000

# Small packets in high frequency (beaconing)
frame.len < 100 && tcp.port == 443
```

### Connection Patterns
```wireshark
# Repeated connection attempts (brute force)
tcp.flags.syn == 1 && tcp.flags.ack == 0

# Connections from unexpected source
ip.src == 192.168.1.0/24 && tcp.dstport == 22

# Connections to unexpected ports (port scanning)
tcp.flags.syn == 1 && (tcp.dstport > 40000 || tcp.dstport < 1024)
```

### Data Exfiltration Patterns
```wireshark
# Large outbound traffic
ip.src == 192.168.1.10 && frame.len > 100000

# DNS TXT record exfiltration
dns.qtype == 16 && dns.qry.name contains "."

# FTP data transfer
ftp-data

# SCP/SSH file transfer
ssh && tcp.port == 22
```

## Combination Filters (Complex Queries)

### Detect Compromised Lateral Movement
```wireshark
# SMB traffic from internal system to multiple destinations
smb && ip.src == 192.168.1.10 && ip.dst != 192.168.1.1
```

### Detect C2 Beaconing
```wireshark
# Regular outbound HTTPS connections to same destination
tls && ip.dst == 45.77.65.211 && frame.time_delta > 30 && frame.time_delta < 120
```

### Detect Data Exfiltration via DNS
```wireshark
# High volume of DNS TXT queries from single source
dns.qtype == 16 && ip.src == 192.168.1.10
```

### Detect Lateral Movement via RDP
```wireshark
# RDP connections from unusual source
tcp.port == 3389 && ip.src == 172.31.10.10 && ip.dst != 192.168.1.1
```

## Filtering Best Practices

1. **Start broad, then narrow** — Filter by IP first, then protocol, then specifics
2. **Use logical operators** — && (AND), || (OR), ! (NOT)
3. **Combine with statistics** — Filter first, then use Statistics menu
4. **Save filters** — Right-click filter bar → Save as Filter
5. **Test on small capture** — Refine logic before applying to large PCAP

---

*For more: Help → Filter Expression Syntax in Wireshark, or visit https://wiki.wireshark.org/DisplayFilters*