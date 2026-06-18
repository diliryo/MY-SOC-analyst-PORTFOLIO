# CASE-004 · Honeypot Deployment & Threat Intelligence

`Status: Documented` · `Category: Threat Intelligence & Adversary Profiling` · `Tools: Kali Linux, Network Segmentation, Log Analysis`

## Overview

This case demonstrates how to deploy isolated honeypots to observe real attacker behavior. By safely exposing fake/low-value systems to the internet, we can collect unfiltered data on how attackers scan, probe, exploit, and persist.

## Objective

Observe and document real-world attacker behavior to:
- Understand common attack techniques (TTPs)
- Build threat actor profiles
- Extract tactics, techniques, and procedures
- Identify trends in exploit tooling and payloads
- Refine defensive signatures and rules

## Honeypot Architecture

### Network Design
```
Internet
    |
[Firewall — Port Forward 22, 80, 443]
    |
[DMZ VLAN — Isolated Network Segment]
    |
  [Honeypot 1: SSH]  [Honeypot 2: HTTP]  [Honeypot 3: DNS]
    |
    └─→ Syslog Server (centralized logging)
    └─→ IDS Sensor (traffic monitoring)

Production Network: BLOCKED from DMZ
```

### Key Design Principles
1. **Isolation** — Honeypots cannot reach production network
2. **Realism** — Honeypots appear to be real systems (OS, services, banners)
3. **Instrumentation** — Every action logged for analysis
4. **Bait** — Attractive targets for attackers (default passwords, old software versions)

## Honeypot Instances

### Honeypot 1: SSH Server (Port 22)
**Purpose:** Monitor credential spray, brute force, and command execution attacks

**Setup:**
- OpenSSH with verbose logging
- Weak default passwords (admin/admin, root/password)
- No real private data, logged commands

**Monitoring:**
- Failed vs. successful login attempts
- Commands executed after compromise
- Tools uploaded/downloaded

### Honeypot 2: HTTP Server (Port 80)
**Purpose:** Observe web-based exploitation and malware distribution

**Setup:**
- Apache with mod_log_forensic
- Vulnerable web application (old CMS, SQL injection vectors)
- Fake admin interface

**Monitoring:**
- Request paths and parameters
- Uploaded files and payloads
- Scanning tools and automated exploitation

### Honeypot 3: DNS Server (Port 53)
**Purpose:** Monitor DNS tunneling and C2 communication

**Setup:**
- BIND with query logging
- Attractive subdomains (internal.company.com, admin.company.com)

**Monitoring:**
- Query patterns and frequency
- Unusual record types (TXT, SRV)
- Data encoding in query names

## Attack Timeline & Findings

### Phase 1: Reconnaissance (Days 1–2)
**Activity:** Network scanning, port enumeration, service banner grabbing

**Indicators:**
- Nmap syn scans from multiple external IPs
- HTTP requests to common paths (/admin, /login, /api)
- DNS zone transfer attempts
- 34 unique external IPs probing honeypots

**Tools Identified:** Nmap, Masscan, curl, wget

### Phase 2: Exploitation Attempts (Days 3–4)
**Activity:** Credential guessing, vulnerability exploitation, payload delivery

**Indicators:**
- 847 SSH brute force attempts against 12 common usernames
- HTTP POST requests with SQL injection payloads
- Failed web shell upload attempts
- Password spray targeting 5 most common passwords: admin, password, 123456, root, test

**Tools Identified:** Hydra (SSH brute force), SQLmap (SQL injection), Weevely (web shell)

### Phase 3: Post-Compromise (Days 5–7)
**Activity:** Command execution, privilege escalation, persistence mechanisms

**Indicators:**
- 13 successful SSH logins using guessed credentials
- Bash history: whoami, id, uname -a, ls -la /home
- Attempts to download and execute scripts from external server
- Scheduled task creation (cron jobs for persistence)

**Payloads Captured:**
- Botnet agent (Mirai variant)
- Cryptominer (XMRig)
- Reverse shell script

## Threat Actor Profile

### Profile: "Mass Exploitation Operator"

**Characteristics:**
- Automated reconnaissance with minimal manual interaction
- Focus on volume over precision (spray & pray)
- Reuses common tools (Hydra, SQLmap, Nmap)
- Rapid exploitation once foothold gained
- Attempts persistence for botnet recruitment

**TTPs (Tactics, Techniques, Procedures):**
- **Reconnaissance:** Active scanning (Nmap)
- **Initial Access:** Credential guessing, default passwords
- **Execution:** Bash command execution via SSH
- **Persistence:** Cron job scheduling
- **Defense Evasion:** Simple obfuscation, script encoding
- **C2:** Direct HTTPS callback to external server

**Indicators of Compromise:**
- External IPs: 13.45.67.89, 198.51.100.45, 203.0.113.78
- Domains: updates.malware-c2.net, payload.botnet-repo.xyz
- Filenames: install.sh, setup.bin, update.py
- Processes: /tmp/miner, /var/tmp/agent

## Key Findings

1. **Attack Volume:** 47 unique external IP addresses probed honeypots over 7 days
2. **Success Rate:** ~1.5% of brute force attempts succeeded (13 of 847)
3. **Top Techniques:** Credential guessing (46%), exploitation (37%), scanning (17%)
4. **Time to Exploit:** Average 3.2 hours from first contact to successful execution
5. **Persistence Attempts:** 100% of successful compromises included persistence mechanism installation

## Defensive Recommendations

### Immediate
- Block observed attacker IPs at firewall
- Update SSH banner to discourage attacks
- Enforce MFA on critical systems
- Monitor for observed malware signatures/file hashes

### Short-term
- Implement account lockout policy (5 failed attempts)
- Deploy fail2ban or similar automated blocking
- Patch identified web application vulnerabilities
- Enable command auditing on critical systems

### Long-term
- Implement network segmentation to limit lateral movement
- Deploy EDR to detect post-compromise activities
- Establish threat intelligence feed subscription
- Regular honeypot deployment to track evolving threats

## Skills Demonstrated

- Honeypot architecture and deployment
- Network segmentation and isolation
- Log analysis at scale
- Attacker TTP extraction
- Threat actor profiling
- Security recommendations based on findings

---

*For network diagram, see [`network-diagram.md`](./network-diagram.md). For detailed attack log analysis, see [`attack-logs-analysis.md`](./attack-logs-analysis.md).*