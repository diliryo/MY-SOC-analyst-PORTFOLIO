# CASE-003 · Network Traffic & Packet Analysis

`Status: Documented` · `Category: Network Forensics` · `Tools: Wireshark, tcpdump, Protocol Analyzers`

## Overview

This case focuses on reading raw network traffic to understand attack patterns and validate SIEM findings. Wireshark analysis provides a ground-truth view of network activity that complements log-based detection.

## Objective

Build the habit of analyzing packet captures (PCAPs) to:
- Validate SIEM alerts with raw evidence
- Reconstruct attacker sessions end-to-end
- Identify command & control (C2) communication
- Detect data exfiltration attempts
- Profile attacker behavior and tools

## Key Skills

- Wireshark filter syntax and workflow
- Protocol analysis (TCP/IP, HTTP, DNS, SMB)
- TCP stream reconstruction
- Baseline vs. anomaly comparison
- Timeline correlation with logs

## Analysis Workflow

### Step 1: Capture Network Traffic
```bash
# Capture on specific interface (example)
tcpdump -i eth0 -w capture.pcap

# Capture with filters (only port 445)
tcpdump -i eth0 -w smb.pcap 'tcp port 445'
```

### Step 2: Open in Wireshark
1. File → Open → capture.pcap
2. Review packet statistics (Protocol Hierarchy)
3. Identify dominant traffic types

### Step 3: Apply Display Filters
```wireshark
# Filter for specific conversation
ip.addr == 45.77.65.211

# Filter for SMB lateral movement
tcp.port == 445 && ip.src == 172.31.10.10

# Filter for DNS exfiltration
dns.qtype == TXT || dns.qtype == SRV
```

### Step 4: Follow TCP Streams
1. Right-click packet → Follow → TCP Stream
2. Analyze conversation content
3. Extract indicators (IPs, domains, commands)

### Step 5: Document Findings
- Screenshot suspicious conversations
- Extract raw data/payloads
- Correlate with SIEM timeline
- Create evidence report

## Case Study: Lateral Movement via SMB

### Traffic Pattern Identified
**Source:** 172.31.10.10 (compromised host)  
**Destination:** 192.168.1.5 (target server)  
**Protocol:** SMB (port 445)  
**Volume:** 247 packets over 3 minutes  

### Wireshark Filter
```wireshark
ip.src == 172.31.10.10 && tcp.port == 445
```

### Findings
- Authentication attempt using `svc_account` credentials
- Successful SMB session established
- Named pipe creation: `\\ADMIN$` (administrative access)
- Suspicious file transfer: `psexec.exe` to remote system
- Command execution via `SVCCTL` remote service

### Evidence
See `screenshots/` folder for annotated Wireshark windows showing SMB handshake, authentication, and command execution.

## Case Study: DNS Tunneling Detection

### Traffic Pattern Identified
**Source:** 192.168.1.10 (internal system)  
**Destination:** 8.8.8.8 (external DNS)  
**Protocol:** DNS (port 53)  
**Anomaly:** Unusual query types (TXT, SRV) in high frequency  

### Wireshark Filter
```wireshark
dns.qtype == TXT || dns.qtype == SRV
```

### Findings
- 147 TXT record queries in 1-hour window (normal: <5)
- SRV queries for non-existent services
- Query names encode exfiltration data
- Response packets contain encoded replies

### Evidence
TXT query example: `exfil-data-base64.internal.domain.com`  
Response: 147 bytes of DNS TXT record data

---

## Skills Demonstrated

- Wireshark filter construction and optimization
- Protocol analysis across TCP/IP stack
- TCP stream reconstruction for conversation review
- Anomaly detection via traffic pattern analysis
- Timeline correlation with other data sources
- Visual documentation of findings

## Key Takeaways

1. **Packet captures tell the truth** — Logs can be tampered with; packet analysis provides ground truth
2. **Filters are essential** — On large captures, targeted filters isolate relevant traffic
3. **Protocol knowledge matters** — Understanding SMB, DNS, HTTP is critical for analysis
4. **Context from multiple sources** — SIEM + packet captures together are more powerful than either alone
5. **Reconstruction reveals intent** — Following TCP streams shows exactly what attacker did

---

*For Wireshark filter cookbook, see [`filter-cookbook.md`](./filter-cookbook.md). For detailed findings, see [`pcap-findings.md`](./pcap-findings.md).*