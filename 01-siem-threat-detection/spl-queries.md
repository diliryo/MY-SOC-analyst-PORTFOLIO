# SPL Query Reference — CASE-001

This document contains commented SPL queries used during the SIEM threat detection lab.

## Detection Queries

### Authentication Brute Force Detection
```spl
index=botsv2 sourcetype="WinEventLog:Security" EventCode=4625 
| stats count as failed_attempts by src_ip, user, dest_ip
| where failed_attempts > 5
| sort - failed_attempts
```

### Lateral Movement via Process Execution
```spl
index=botsv2 sourcetype="WinEventLog:Security" EventCode=4688
| eval process_name=mvindex(split(process, "\\"), -1)
| where process_name IN ("powershell.exe", "cmd.exe", "psexec.exe", "schtasks.exe")
| stats count by process, parent_process, dest_ip
| where count > 3
```

### Data Exfiltration via DNS Tunneling
```spl
index=botsv2 sourcetype="dns"
| where query_type NOT IN ("A", "AAAA", "CNAME", "MX")
| stats count as unusual_queries by src_ip, query_type
| where unusual_queries > 10
| sort - unusual_queries
```

## Monitoring Queries

### HTTP Traffic Volume by Method (Hourly)
```spl
index=botsv2 sourcetype="access_combined"
| timechart span=1h count by http_method limit=5
```

### Top Source/Destination IP Pairs
```spl
index=botsv2 sourcetype="access_combined"
| stats count as connection_count by src_ip, dest_ip, http_status
| where connection_count > 100
| sort - connection_count
```

### Geographic Distribution of Threats
```spl
index=botsv2 sourcetype="access_combined" http_status!=200
| geostats count by src_ip, http_status
```

### Status Code Distribution
```spl
index=botsv2 sourcetype="access_combined"
| stats count by http_status
| where count > 10
```

## Dashboard Queries

### Widget 1: Event Count (Single Value)
```spl
index=botsv2
| stats count
```

### Widget 2: Timeline of Activity
```spl
index=botsv2 sourcetype="access_combined"
| timechart span=1d count
```

### Widget 3: Top 10 IPs by Traffic
```spl
index=botsv2 sourcetype="access_combined"
| stats count by src_ip
| sort - count
| head 10
```

---

*All queries assume BOTSv2 is loaded in index `botsv2`. Adjust sourcetype/index names as needed for your environment.*