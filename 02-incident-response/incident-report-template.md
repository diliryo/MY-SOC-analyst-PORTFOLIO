# Incident Report Template

**Use this template for formal incident documentation.**

---

## INCIDENT REPORT

**Report Date:** [YYYY-MM-DD]  
**Incident ID:** [IR-2024-XXX]  
**Severity:** [Critical | High | Medium | Low]  
**Status:** [Open | Closed | Escalated]

---

## EXECUTIVE SUMMARY

*Brief 2–3 sentence overview for non-technical stakeholders. Include: what happened, impact, current status.*

> On [DATE], we detected [BREACH TYPE] affecting [NUMBER] systems. [IMPACT STATEMENT]. Current status: [MITIGATION ACTION].

---

## AFFECTED ASSETS

| Asset Type | Count | Examples |
|---|---|---|
| Hosts | X | server1, server2, ... |
| Users | X | user@company.com, ... |
| Data | X | Customer database, financial records |
| Systems | X | Email, file server, CRM |

---

## INDICATORS OF COMPROMISE (IoCs)

### IPs
- 45.77.65.211 (external threat actor)
- 172.31.10.10 (internal compromised host)

### Domains
- suspicious.domain.com
- exfil.domain.net

### File Hashes
- MD5: [hash]
- SHA-256: [hash]

### Processes
- powershell.exe (parent: svchost.exe)
- cmd.exe (parent: explorer.exe)
- psexec.exe (parent: svc_account)

### Accounts
- svc_account (compromised)
- admin (privilege escalation attempt)

---

## TIMELINE

| Date & Time | Event | Severity | Evidence |
|---|---|---|---|
| Aug 9, 14:22 UTC | Initial recon scan detected | Medium | 47 failed login attempts (Event ID 4625) |
| Aug 11, 09:30 UTC | **Spike in activity** | High | 8,847 events in 2 hours (geostats map) |
| Aug 11, 11:15 UTC | Account compromise confirmed | Critical | Successful login from 45.77.65.211 |
| Aug 12, 08:15 UTC | **Lateral movement detected** | Critical | psexec.exe execution across 5 hosts |
| Aug 14, 08:00 UTC | **Data exfiltration begins** | Critical | Large HTTP POST to exfil domain |
| Aug 14, 18:30 UTC | Persistence mechanism set | High | Scheduled task created (schtasks.exe) |
| Aug 15, 10:00 UTC | Incident detected & contained | N/A | All systems isolated |

---

## ATTACK METHODOLOGY

### Phase 1: Reconnaissance (Aug 9)
Attacker began with network reconnaissance, attempting to identify valid user accounts through brute force password guessing against the domain controller.

**Evidence:**
- 47 failed authentication attempts in 30-minute window
- Attacks targeting known service account names
- External source IP (45.77.65.211) with no legitimate business use

### Phase 2: Initial Compromise (Aug 11)
Attacker successfully compromised the `svc_account` service account, possibly by guessing weak password or through credential stuffing.

**Evidence:**
- Successful authentication from 45.77.65.211 → svc_account login
- Spike in event volume immediately after successful login
- PowerShell execution with local system privileges

### Phase 3: Lateral Movement (Aug 12–13)
Using the compromised service account, attacker moved laterally across the network to establish additional footholds.

**Evidence:**
- psexec.exe process execution from svc_account
- Connections to port 445 (SMB) across multiple subnets
- Privilege escalation to local administrator on 5 target systems
- Scheduled tasks created for persistence

### Phase 4: Data Exfiltration (Aug 14)
Attacker executed final stage: extracting sensitive data to external servers.

**Evidence:**
- Large HTTP POST requests (>5MB) to exfil.domain.net
- Unusual DNS query patterns (TXT records, SRV queries)
- Database query activity on systems containing customer data

---

## IMPACT ASSESSMENT

### Business Impact
- **Systems Down:** [X hours]
- **Users Affected:** [X]
- **Data Exposed:** [X records]
- **Financial Impact:** [$ estimate or TBD pending discovery]

### Technical Impact
- 5 systems compromised and isolated
- 1 service account password reset
- 47 failed authentication attempts against domain controller
- ~15GB of data exfiltrated to external servers

### Reputational Impact
- Customer notification required (if PII exposed)
- Regulatory reporting obligations (GDPR, HIPAA, etc.)
- Press/media consideration

---

## ROOT CAUSE ANALYSIS

### Primary Cause
**Weak password on service account** — The `svc_account` had a weak, guessable password that was compromised through brute force or credential stuffing.

### Contributing Factors
1. No MFA on service accounts
2. Excessive permissions granted to service account (should have been least-privilege)
3. Lack of network segmentation (no isolation between subnets)
4. Insufficient monitoring of SMB lateral movement attempts
5. Slow SIEM alert response (4 days before detection)

---

## REMEDIATION ACTIONS

### Immediate (Completed)
- ✅ Reset svc_account password
- ✅ Block attacker IP at firewall (persistent rule)
- ✅ Isolate 5 compromised systems from network
- ✅ Kill all suspicious processes on affected systems

### Short-term (24–48 hours)
- ⏳ Run full antivirus scan on all systems
- ⏳ Audit all scheduled tasks for persistence mechanisms
- ⏳ Review SMB connection logs for additional lateral movement
- ⏳ Reset passwords for all accounts that logged in during incident window
- ⏳ Perform disk imaging for forensic analysis

### Long-term (1–2 weeks)
- Implement Multi-Factor Authentication (MFA) on all service accounts
- Deploy Network Access Control (NAC) to enforce least-privilege access
- Implement network segmentation (VLANs) to isolate critical assets
- Deploy EDR (Endpoint Detection & Response) agent to all systems
- Strengthen SIEM alerting for SMB lateral movement
- Conduct security awareness training on credential management

---

## LESSONS LEARNED

1. **Service account hygiene matters** — Treat service accounts like regular accounts: strong passwords, MFA, regular rotation
2. **Early detection is critical** — 4-day detection window is too long; we need sub-hour alerting for lateral movement
3. **Segmentation prevents spread** — Without network isolation, attacker was able to freely move between subnets
4. **Documentation enables response** — Clear incident tickets and timelines accelerated our investigation

---

## SIGN-OFF

**Incident Lead:** [Name]  
**Date Completed:** [YYYY-MM-DD]  
**Approver:** [Manager Name]  
**Distribution:** [Stakeholders: CEO, CFO, Legal, etc.]

---

*Report Classification: [Internal | Confidential | Public]*  
*Next Review Date: [YYYY-MM-DD]*