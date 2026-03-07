# DNS Threat Hunting with Splunk SIEM: Analyzing DNS Logs for Security Insights

[![GitHub stars](https://img.shields.io/github/stars/umang220/DNS-Threat-Hunting-Splunk-Analysis)](https://github.com/umang220/DNS-Threat-Hunting-Splunk-Analysis)
[![GitHub license](https://img.shields.io/github/license/umang220/DNS-Threat-Hunting-Splunk-Analysis)](https://github.com/umang220/DNS-Threat-Hunting-Splunk-Analysis/blob/main/LICENSE)

## 🚀 Project Overview
This project demonstrates how to ingest, analyze, and visualize DNS log files using **Splunk SIEM** for threat hunting. Key focus: Detecting anomalies like suspicious domains, high-volume queries, tunneling, DGA (Domain Generation Algorithms), and off-hours activity.

**Why DNS Analysis?** DNS is often the first step in attacks (e.g., C2 communication, data exfiltration). This setup helps SOC analysts identify threats early.

### 🎯 Objectives
- Parse and index DNS logs (BIND, Windows, or Cisco Umbrella format).
- Build SPL queries for anomaly detection.
- Create an interactive dashboard for real-time monitoring.
- Simulate threat scenarios and generate alerts.

## 🛠️ Tech Stack
- **Splunk Enterprise** (SIEM tool)
- **SPL (Search Processing Language)** for queries
- **Markdown** for documentation
- Sample data: Synthetic DNS logs (no real data shared for privacy)

## 📊 Step-by-Step Implementation

### 1. Data Ingestion
- Uploaded sample DNS logs to Splunk via `Add Data` > `Upload`.
- Configured sourcetype: `dns` with custom field extractions (e.g., `query` via regex: `r"(?P<query>[^.]+)\..*"`).
- Indexed under `index=dns_logs`.

### 2. Key SPL Queries
Here are the core searches I built:

#### Top Queried Domains (High-Volume Indicator)
```spl
index=dns_logs sourcetype=dns | stats count by query | sort - count | head 10 | rename count as "Query_Volume"

#### Long Subdomains (DNS Tunneling Detection)
```spl
index=dns_logs sourcetype=dns 
| eval length=len(query) 
| where length > 50 
| stats count by query, src_ip 
| sort -count

#### Off-Hours Query Spikes (Unusual Timing Detection)
```spl
index=dns_logs sourcetype=dns 
| eval hour=strftime(_time, "%H") 
| where hour < "06" OR hour > "22" 
| timechart span=1h count by src_ip

#### Rare / Unique Domains per IP (DGA / Beaconing Detection)
```spl
index=dns_logs sourcetype=dns 
| stats dc(query) as unique_domains by src_ip 
| where unique_domains > 100 
| sort -unique_domains

### 3. Dashboard Creation
Built a custom Splunk dashboard for DNS threat monitoring:
- Panels included: Top domains (bar chart), Query volume over time (line chart), Off-hours spikes, Long subdomains table, Top talkers by IP.
- Used above SPL queries as data sources for panels.
- Dashboard helps quick visual threat hunting.

![DNS Threat Hunting Dashboard]<image-card alt="Splunk DNS Logs Search Example" src="images/splunk-dnslogs-search-example.png" ></image-card>
*(Screenshot of the full dashboard – upload your Splunk export soon)*
