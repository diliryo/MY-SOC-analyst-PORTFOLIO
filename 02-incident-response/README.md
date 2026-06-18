# CASE-002 · Incident Triage, Ticketing & Reporting

`Status: Documented` · `Category: Incident Response` · `Focus: Alert Management, Triage, Reporting`

## Overview

This case demonstrates the critical skills that happen *after* detection — how to decide which alerts matter, track them systematically, and communicate findings to stakeholders. The lab uses alerts from CASE-001 to practice real-world incident response workflows.

## Objective

Transform high-volume security alerts into prioritized, documented incidents that can be handed off to incident response teams or remediation efforts.

## Incident Management Workflow

### Phase 1: Alert Ingestion
- Receive raw alerts from SIEM dashboard
- Log each alert in a ticketing system
- Capture: timestamp, source IP, affected asset, alert type, raw event data

### Phase 2: Triage & Prioritization
- Classify alert severity (Critical/High/Medium/Low)
- Assess business impact (which assets/users affected?)
- Determine urgency (SLA-based)
- Assign to appropriate team

### Phase 3: Investigation
- Gather additional context from SIEM queries
- Correlate with other alerts
- Extract indicators of compromise (IoCs)
- Document findings and timeline

### Phase 4: Reporting
- Write incident summary for stakeholders
- Create formal incident report
- Document remediation actions taken
- Update ticket status and close

## Triage Methodology

### Severity Assessment Framework

| Severity | Criteria | SLA | Example |
|---|---|---|---|
| **Critical** | Confirmed compromise, data exfiltration, or widespread impact | 1 hour | Ransomware execution, mass credential theft |
| **High** | Strong indicators of attack, limited scope, containment possible | 4 hours | Lateral movement, privilege escalation attempt |
| **Medium** | Suspicious activity, low confirmation, single user/asset | 24 hours | Failed brute force, policy violation |
| **Low** | Informational alert, routine activity detected | 72 hours | Failed login from known IP, scan detection |

### Prioritization Matrix

**Severity × Impact = Priority**

- **Critical + Widespread** → P1 (Immediate action)
- **Critical + Limited** → P2 (Within 1 hour)
- **High + Any Impact** → P3 (Within 4 hours)
- **Medium + Any Impact** → P4 (Within 24 hours)
- **Low** → P5 (Background work)

## Sample Incident Response

### Alert #001: Failed Brute Force Attack

**Detected:** August 9, 14:22 UTC  
**Source IP:** 45.77.65.211  
**Target:** Domain controllers  
**Event Count:** 47 failed logins in 30 minutes  

**Triage Assessment:**
- Severity: **HIGH** (credential attack on critical asset)
- Impact: **CONTAINED** (attack unsuccessful, no successful login)
- Priority: **P3** (investigate within 4 hours)

**Investigation:**
- Confirmed external source IP with no legitimate business use
- Password policy prevented account lockout
- No successful authentications from this IP

**Action:**
- Block IP at firewall (persistent rule)
- Reset passwords for targeted accounts
- Enable MFA on domain accounts
- Close ticket: Mitigated

---

### Alert #002: Lateral Movement Detected

**Detected:** August 12, 08:15 UTC  
**Source IP:** 172.31.10.10 (internal)  
**Activity:** SMB connections to 5+ systems  
**Process:** psexec.exe  

**Triage Assessment:**
- Severity: **CRITICAL** (confirmed lateral movement)
- Impact: **WIDESPREAD** (multiple systems affected)
- Priority: **P1** (immediate escalation)

**Investigation:**
- Trace back to compromised account: `svc_account`
- Account used for scheduled backup job (explains SMB access)
- Evidence of privilege escalation to local admin on target systems
- Timeline suggests compromise started August 11

**Action:**
- ESCALATE to IR team immediately
- Isolate affected systems from network
- Revoke compromised account credentials
- Initiate forensic analysis
- Status: Escalated → Forensics Team

---

## Skills Demonstrated

- Alert triage and severity assessment
- Incident ticketing and tracking
- Correlating multiple data sources
- Timeline reconstruction
- Incident report writing
- Stakeholder communication
- Escalation decision-making

## Key Takeaways

1. **Not all alerts are equal** — Triage correctly to avoid alert fatigue
2. **Context is critical** — Know your environment, asset criticality, and normal baselines
3. **Documentation matters** — Future analysts depend on clear, complete incident records
4. **Speed + accuracy** — Balance fast response with thorough investigation
5. **Communication is key** — Translate technical findings for non-technical stakeholders

---

*For incident report template, see [`incident-report-template.md`](./incident-report-template.md). For sample tickets, see [`sample-tickets.md`](./sample-tickets.md).*