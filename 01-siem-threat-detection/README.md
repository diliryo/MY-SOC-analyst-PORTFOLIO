# CASE-001 · SIEM Threat Detection & Dashboard Build

`Status: Documented` · `Category: Detection & Monitoring` · `Tools: Splunk Enterprise, BOTSv2 Dataset, SPL`

## Overview

This case covers standing up a working SIEM environment and learning to turn raw security logs into actionable monitoring. The lab uses **BOTSv2 (Boss of the SOC v2)** — a publicly available dataset that simulates a real breach scenario with 68M+ events across multiple log sources.

## Lab Environment

| Component | Detail |
|---|---|
| SIEM Platform | Splunk Enterprise 9.x |
| Dataset | BOTSv2 |
| Indexes Used | `botsv2`, `botsv2_windows`, `botsv2_web` |
| Key Sourcetypes | `WinEventLog:Security`, `stream:http`, `suricata`, `access_combined` |
| Deployment | Local VM / Docker instance |

## Methodology

1. **Ingest & Explore** — Loaded BOTSv2 into Splunk, ran broad searches to understand sourcetype/field structure
2. **Hunt with SPL** — Wrote searches to surface indicators of compromise (IoCs) across the simulated environment
3. **Build Dashboards** — Translated searches into permanent dashboard panels with visualizations
4. **Rehearse & Present** — Practiced explaining findings to non-technical stakeholders

## Key SPL Queries

### Query 1: Suspicious Authentication Activity
```spl
index=botsv2 sourcetype="WinEventLog:Security" EventCode=4625
| stats count by src_ip, dest_ip, user
| where count > 5
| sort - count
```
*Detects failed login attempts indicating credential spray or brute force attacks.*

### Query 2: Anomalous Process Execution
```spl
index=botsv2 sourcetype="WinEventLog:Security" EventCode=4688
| stats count by process, parent_process
| where process IN ("powershell.exe", "cmd.exe", "schtasks.exe")
```
*Identifies suspicious process spawning patterns commonly seen in lateral movement.*

### Query 3: Unusual DNS Query Patterns
```spl
index=botsv2 sourcetype="dns" NOT query_type="A" 
| stats count by query, query_type, src_ip
| where count > 20
```
*Surfaces DNS tunneling or data exfiltration attempts via unusual query types.*

### Query 4: Web Traffic Over Time
```spl
index=botsv2 sourcetype="access_combined" 
| timechart span=1h count by http_method
```
*Visualizes HTTP request patterns to identify anomalous spikes or methods.*

## Key Findings

- **Total Events Processed:** 68.8M+ events indicating high-volume log ingestion
- **Top Malicious IPs:** Two source IPs (172.31.10.10, 45.77.65.211) responsible for ~19,000 events each
- **Geospatial Distribution:** Largest threat concentration from Northern Europe/Scandinavia region
- **HTTP Anomalies:** Two distinct traffic spikes (Aug 11 & Aug 14) with GET request dominance
- **Status Codes:** 99%+ successful responses (200 OK), with small error code subset indicating targeted attacks

## Dashboard Overview

**Panels Included:**
- Geographic heat map of source IPs (geostats)
- HTTP method distribution over time (timechart)
- Top 10 source/destination IP pairs (bar chart)
- HTTP status code breakdown (pie chart)
- Real-time event count (single value)
- Protocol hierarchy table

📁 See `screenshots/` folder for dashboard exports and visualizations.

## Skills Demonstrated

- SPL query writing (search, stats, timechart, eval, geostats)
- Log correlation across multiple sourcetypes
- Dashboard design and visualization selection
- Technical communication to non-technical audiences
- SIEM operational readiness

## Reflection

**Key Achievements:**
- Successfully ingested and normalized 68M+ events from diverse log sources
- Designed a dashboard that provides both tactical (IoC detection) and strategic (trend analysis) insights
- Demonstrated ability to go from raw logs → actionable intelligence
- Practiced translating technical findings into stakeholder-ready narratives

**Lessons Learned:**
- Geospatial + temporal correlation reveals patterns that individual searches miss
- Dashboard simplicity matters — too many panels overwhelms users
- Understanding log structure (sourcetypes, fields) is critical before writing queries
- Real-time dashboards require well-tuned queries to avoid timeout issues at scale

---

*For detailed SPL queries, see [`spl-queries.md`](./spl-queries.md). For findings and IoC indicators, see [`findings.md`](./findings.md).*