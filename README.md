# network-attack-detection-lab
Detecting real network attacks using Suricata IDS, Splunk SIEM, Kali Linux. Covers TCP/UDP attack generation, packet analysis with tshark, and log investigation with Splunk SPL queries.

A home SOC lab built to generate, capture, and investigate 
real network attacks using industry-standard security tools.

## Lab Environment

| Machine | OS | Role |
|---|---|---|
| Kali Linux | Kali Rolling | Attacker |
| Ubuntu + Suricata | Ubuntu 22.04 | IDS sensor + Victim |
| Ubuntu + Splunk | Ubuntu 22.04 | Splunk SIEM |

## Tools Used

- **Suricata** — Network IDS that monitors traffic and 
  fires alerts when attack patterns are detected
- **Splunk** — SIEM that receives Suricata logs via 
  Universal Forwarder for investigation and searching
- **tshark** — Command line packet analyzer for 
  inspecting pcap files at packet level
- **tcpdump** — Captures live traffic to pcap files 
  during attacks
- **Nmap** — Port scanner run from Kali to generate 
  reconnaissance traffic

## Investigations

| # | Attack | Protocol | Status |
|---|---|---|---|
|  | TCP Port Scan | TCP | ✓ Complete |

## What This Lab Demonstrates

- Configuring Suricata IDS rules and network variables
- Capturing live network traffic with tcpdump
- Analyzing packets with tshark display filters
- Forwarding Suricata logs to Splunk via 
  Universal Forwarder
- Writing Splunk SPL queries to detect attack patterns
- Reading and interpreting Suricata eve.json alerts
- Distinguishing malicious traffic from 
  legitimate traffic
