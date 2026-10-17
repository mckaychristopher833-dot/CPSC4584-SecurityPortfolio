# Week 7: Network Traffic Anomaly During Night Shift
**Course:** CPSC 4584 | Special Topics in Information Security  
**Date:** October 12, 2026  
**Analyst:** Christopher McKay  
**Case ID:** NET-2026-1012-001 (related: FOR-2026-1005-001)  

---

## Incident Summary

During the October 3–5, 2026 custody gap window, network flow logs revealed a 2.46 MB outbound transfer and continuous 4-minute beaconing from Marcus Webb’s laptop (`10.4.2.47`) to an unauthorized external IP (`185.220.101.47`). This network activity closely correlates with the file modification and deletion timeline established during the Week 6 disk forensic analysis of Webb's device, alongside additional anomalous traffic detected on billing and radiology workstations.

---

## Network Flow Analysis

| Flow | Source | Destination | Classification | Finding |
|------|--------|-------------|----------------|---------|
| 1 | Webb laptop (`10.4.2.47`) | `185.220.101.47` | Unauthorized Destination / Exfiltration | Supports an outbound data transfer of 2,457,814 bytes (~2.46 MB) over HTTPS/443 at 11:48 PM on Oct 3, shortly after `insurance_export_final.xlsx` was modified. |
| 2 | Webb laptop (`10.4.2.47`) | `185.220.101.47` | C2 Beaconing Pattern | Supports beaconing behavior due to highly regular 4-minute connection intervals and identical 312-byte payloads from 11:15 PM (Oct 3) to 6:42 AM (Oct 4). |
| 3 | EHR prod server (`10.4.1.10`) | AWS (`52.94.228.167`) | Authorized Baseline Traffic | Expected nightly backup traffic; represents a scheduled 14.2 MB HTTPS backup transfer to the vendor between 2:00 AM and 2:18 AM on Oct 4. |
| 4 | Radiology WS (`10.4.3.22`) | `8.8.8.8` | DNS Query Anomaly | Anomaly characterized by an unusual volume of 23 DNS queries (847 bytes) within an 8-minute window (2:14 AM–2:22 AM on Oct 4). |
| 5 | Billing WS (`10.4.3.88`) | `203.0.113.99` | Unapproved Destination / Potential Beaconing | 9 repeated 4,200-byte HTTP connections at 4-minute intervals (3:47 AM–4:19 AM) to an unapproved IP are consistent with automated/beaconing patterns, but require payload review to confirm C2. |

---

## Disk and Network Evidence Correlation

* **Disk (Week 6):** `insurance_export_final.xlsx` modified at 11:47 PM on Oct 3, 2026; file deleted at 11:52 PM on Oct 3, 2026.
* **Network (Week 7):** Flow 1 records a 2,457,814-byte (~2.46 MB) outbound HTTPS transfer to `185.220.101.47` between 11:48 PM and 11:50 PM on Oct 3, 2026.
* **Correlation:** The timeline establishes a coherent **modify → transfer → delete** sequence within a 5-minute window during Webb's custody gap.
* **Limitation:** Flow logs summarize connection metadata (IPs, ports, byte counts) but do not record encrypted application payloads. Without full packet capture (pcap) or endpoint decryption logs, flow data alone cannot confirm that `insurance_export_final.xlsx` was the specific file transmitted.

---

## Network Artifact Inspection Commands Used

| Command | Purpose |
|---------|---------|
| `curl -I https://example.com` | Observed Layer 7 application headers (`HTTP/2 200`, `content-type`, server metadata) without downloading the response body. |
| `curl -v https://example.com` | Displayed network connection setup, server IP resolution (`93.184.215.14`), TCP port 443 handshake, and the negotiated TLS 1.3 encryption exchange. |
| `curl -s https://example.com \| xxd -l 64` | Analyzed the raw initial bytes of the payload in hex/ASCII, confirming the HTML document signature (`<!doctype html>`). |
| `curl -s -w "remote_ip: %{remote_ip}\n..." -o /dev/null https://example.com` | Extracted session metrics (remote IP, port, bytes downloaded, connection time) that directly map to standard network flow fields. |

---

## Escalation Summary

**Confirmed Findings:**  
1. Webb's laptop (`10.4.2.47`) completed an unauthorized 2.46 MB outbound HTTPS transfer to `185.220.101.47` at 11:48 PM on Oct 3, followed by periodic 312-byte connections every 4 minutes.  
2. Disk forensic evidence confirms file modification at 11:47 PM and deletion at 11:52 PM on the same device.  
3. Destination IP `185.220.101.47` resides in the same `/24` subnet as IP addresses documented in the billing portal credential stuffing attack (`INC-2026-0928-001`).  
4. Billing (`10.4.3.88`) and Radiology (`10.4.3.22`) workstations demonstrated secondary traffic anomalies during the same night shift.

**Unresolved Questions:**  
1. The encrypted payload prevents direct confirmation of the exact data transferred during Flow 1.  
2. Subnet correlation links destination hosts to prior threat infrastructure but does not definitively prove single-actor attribution.

**Escalation & Next Steps:**  
In compliance with analyst constraint rules, no unilateral blocking or containment has been executed. It is recommended that SOC leadership escalate this matter to the Incident Response Lead to authorize:  
* Preservation and forensic imaging of workstations `10.4.3.22` and `10.4.3.88`.  
* Coordinated network isolation of affected endpoints.  
* Requesting host-based endpoint detection (EDR) and memory artifacts to inspect decrypted session buffers.

---
*CPSC 4584 | Governors State University | Fall 2026*
