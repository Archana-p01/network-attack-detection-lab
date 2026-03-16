# Lab Setup

## Network Architecture

All three machines are on the same internal network.
No traffic leaves the lab environment.
```
Kali Linux (192.168.1.x)
    │
    │  generates attacks
    ▼
Ubuntu + Suricata (192.168.1.x)
    │
    │  monitors all traffic
    │  fires alerts
    │  forwards logs via Splunk Universal Forwarder
    ▼
Ubuntu + Splunk (192.168.1.x)
    │
    │  receives logs on port 9997
    │  stores and indexes eve.json
    │  used for investigation and SPL queries
```

## Machine Details

### Kali Linux — Attacker
- Role: Generates attacks against target machines
- Tools used: Nmap
- IP: 192.168.1.x

### Ubuntu + Suricata — IDS Sensor
- Role: Monitors all network traffic and fires alerts
- Interface monitored: enp0s3
- Alert log: /var/log/suricata/fast.log
- Full event log: /var/log/suricata/eve.json
- IP: 192.168.1.x

### Ubuntu + Splunk — SIEM
- Role: Receives and indexes Suricata logs
- Receives logs on port 9997
- IP: 192.168.1.x

## Suricata Configuration Changes Made

### 1. Set correct network interface
File: /etc/suricata/suricata.yaml
```
af-packet:
  - interface: enp0s3
```

### 2. Set HOME_NET to lab machine IP
File: /etc/suricata/suricata.yaml
```
HOME_NET: "[192.168.1.x]"
```

### 3. Changed EXTERNAL_NET to any
Required because Kali is on the same internal
network as the sensor. Without this change
Suricata treats Kali as internal traffic and
scan rules do not fire.

File: /etc/suricata/suricata.yaml
```
EXTERNAL_NET: "any"
```

### 4. Uncommented Nmap scan detection rule
Nmap detection rules are commented out by default
in Suricata. Had to manually enable them.

File: /var/lib/suricata/rules/suricata.rules

Removed the # from the start of this rule:
```
alert tcp $EXTERNAL_NET any -> $HOME_NET any
(msg:"ET SCAN NMAP -sS window 2048";
fragbits:!D; dsize:0; flags:S,12; ack:0;
window:2048; threshold: type both, track by_dst,
count 1, seconds 60; classtype:attempted-recon;
sid:2000537; rev:8;)
```

## Splunk Universal Forwarder Setup

Installed on the Suricata machine to ship
eve.json logs to Splunk.

### Add forward server
```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.1.x:9997
```

### Monitor eve.json
```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/suricata/eve.json -index suricata -sourcetype suricata
```

### Verify everything is connected
```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server
sudo /opt/splunkforwarder/bin/splunk list monitor
```

## Issues Encountered and Fixed

### Issue 1 — No alerts firing from Nmap scan
**Cause:** Nmap scan rules were commented out in
suricata.rules by default.

**Fix:** Manually removed # from ET SCAN NMAP rule.
```bash
sudo nano /var/lib/suricata/rules/suricata.rules
# removed # from ET SCAN NMAP -sS window 2048 rule
sudo systemctl restart suricata
```

### Issue 2 — Wireshark no display
**Cause:** Suricata machine has no GUI so Wireshark
could not open.

**Fix:** Used tshark instead — command line version
of Wireshark, does everything in the terminal.
```bash
sudo apt install tshark -y
tshark -r capture.pcap -Y "tcp.flags.syn==1"
```
