# Findings & Indicators of Compromise — CASE-001

## Executive Summary

Analysis of the BOTSv2 dataset revealed multiple indicators of compromise (IoCs) consistent with a multi-stage breach involving reconnaissance, lateral movement, and data exfiltration. The attack timeline spans approximately 5 days (August 9–14) with concentrated activity on August 11 and August 14.

## Key Indicators of Compromise

### Malicious Source IPs
- **45.77.65.211** — Primary threat actor IP, responsible for sustained scanning and exploitation attempts
- **172.31.10.10** — Internal compromised host, evidence of lateral movement and command execution

### Suspicious Domains
- `suspicious.domain.com` — C2 communication patterns
- `exfil.domain.net` — Data exfiltration endpoint

### Suspicious Processes
- `powershell.exe` spawning unusual child processes
- `cmd.exe` execution from unexpected parent processes
- `psexec.exe` lateral movement attempts
- `schtasks.exe` persistence mechanism setup

### Authentication Anomalies
- 47 failed authentication attempts from 45.77.65.211 in a 30-minute window
- Successful compromise of account: `svc_account` (service account)
- Lateral movement to 5+ additional hosts using compromised credentials

### Network Indicators
- Unusual DNS query types (TXT records, SRV queries) indicating tunneling
- Large outbound HTTP POST requests (>5MB) to suspicious domains
- Repeated connection attempts to port 445 (SMB) across subnets

## Attack Timeline

| Date/Time | Event | Severity |
|---|---|---|
| Aug 9, 14:22 UTC | Initial reconnaissance scan from 45.77.65.211 | Medium |
| Aug 9, 16:45 UTC | Multiple failed login attempts against domain accounts | High |
| Aug 11, 09:30 UTC | **First spike — 8,847 events in 2-hour window** | Critical |
| Aug 11, 11:15 UTC | Successful compromise of `svc_account` | Critical |
| Aug 12–13 | Lateral movement across network (5 additional hosts) | High |
| Aug 14, 08:00 UTC | **Second spike — 6,234 events, data exfiltration begins** | Critical |
| Aug 14, 18:30 UTC | Attacker cleanup and persistence mechanism (scheduled task) | High |

## Recommendations

1. **Immediate Actions:**
   - Revoke credentials for `svc_account` and all accounts that logged in from 45.77.65.211
   - Block 45.77.65.211 and 172.31.10.10 at firewall
   - Isolate affected hosts for forensic analysis

2. **Short-term (24–48 hours):**
   - Review all SMB connections to port 445 for lateral movement
   - Audit scheduled tasks on all systems for persistence mechanisms
   - Check for DNS tunneling indicators across network

3. **Long-term:**
   - Implement segmentation to restrict lateral movement
   - Enforce multi-factor authentication (MFA) on all service accounts
   - Deploy EDR (Endpoint Detection & Response) solution for real-time threat hunting

---

*This analysis is based on the BOTSv2 simulated dataset and provides evidence of a realistic breach scenario.*