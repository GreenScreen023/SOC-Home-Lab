# Network Forensics — Baseline vs Attack Traffic Analysis

## Overview
Packet capture analysis using tshark (Wireshark CLI) to establish a 
normal traffic baseline and compare it against active attack traffic. 
This demonstrates how network forensics complements SIEM log analysis 
— Wazuh tells you WHAT happened, Wireshark tells you HOW it happened 
at the packet level.

## Tool Used
- tshark (Wireshark command-line interface) installed on Wazuh SIEM VM
- Captures saved as .pcap files for offline analysis

## Baseline Capture — Normal Network Traffic
Captured with all three VMs idle. No attacks, no commands, just normal 
background traffic over approximately 30 minutes.

### Protocols Observed (Normal)
- ETH — Standard ethernet framing on all packets
- ARP — Machines announcing their presence on the network
- IP/TCP — Wazuh agent heartbeats to the manager
- UDP/mDNS — Machine hostname announcements
- SSDP — Background service discovery
- IPv6/ICMPv6 — Standard IPv6 neighbor discovery

### Key Metric — Normal
- RST (Reset) packets: 0
- SSH packets: 0

## Attack Capture — Active Attack Traffic
Captured during simultaneous Nmap SYN scan and Hydra SSH brute force 
attempt against the hardened Ubuntu target.

### Key Findings
| Metric | Baseline | Under Attack | Difference |
|--------|----------|--------------|------------|
| RST packets | 0 | 2,043 | +2,043 |
| SSH packets | 0 | 70 | +70 |

## What the Numbers Mean

### 2,043 RST Packets = Nmap Port Scan Signature
A RST (Reset) packet means a connection was abruptly terminated without 
a proper goodbye. Normal traffic has zero RSTs. During the Nmap scan, 
Kali sent a SYN to every port, received a SYN-ACK, then immediately 
sent RST — "never mind." 2,043 times. That flood of RST packets is 
the unmistakable network signature of a port scan. No logs needed — 
the packets tell the story.

### 70 SSH Packets = Brute Force Attempt (Failed)
Zero SSH packets in baseline means no one was connecting via SSH during 
normal operation. 70 SSH packets during the attack window means Hydra 
was hammering the SSH port. Note: the brute force FAILED this time 
because SSH password authentication was disabled during hardening. The 
attacker got 70 packets of rejection.

## Correlation With Wazuh SIEM
The same attack generated alerts in Wazuh simultaneously:
- Rule 5760 — SSH authentication failures (brute force detected)
- Network layer confirmed: 70 SSH packets with no successful handshake

Two independent detection layers. Same attack. Same timeline. 
Neither one alone tells the complete story. Together they do.

## Key Takeaway
Baseline captures are essential. Without knowing what normal looks 
like, anomalies are invisible. The 2,043 RST packets only mean 
something because the baseline showed zero. Establishing baselines 
is a day-one task for any SOC analyst joining a new environment.

## Files
- baseline-normal.pcap — Normal traffic capture
- baseline-summary.txt — Protocol hierarchy, normal traffic  
- attack-capture.pcap — Attack traffic capture
- attack-summary.txt — Protocol hierarchy, attack traffic
