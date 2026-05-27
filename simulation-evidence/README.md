# Lab Screenshots & Evidence

## Wazuh Installation Complete
The Wazuh SIEM dashboard fully installed and accessible via browser. 
The install script confirms all three components running: manager, 
indexer, and dashboard. Admin credentials generated at completion.

## VMware Network Configuration
VMware Virtual Network Editor showing VMnet1 configured as a host-only 
network on the 192.168.x.x/24 subnet. DHCP disabled — all three VMs 
assigned static addresses manually. No traffic reaches the real LAN.

## Brute Force Attack — Live Detection
Split screen showing the attack and defense simultaneously. Left side: 
Hydra running against the target's SSH service, cracking the password 
in under 10 seconds. Right side: Wazuh Events tab showing Rule 5760 
(SSH authentication failures) firing in real time, immediately followed 
by Rule 5715 (successful login after brute force). The detection 
happened within seconds of the attack completing.

## Privilege Escalation Confirmed
Terminal output showing the before and after of the SUID bash exploit. 
Before: uid=1000(labuser) — standard unprivileged user. After running 
/bin/bash -p: euid=0(root) — full administrator access achieved from 
a standard account. whoami confirms: root.

## File Integrity Monitor — Crontab Modification Detected
Wazuh File Integrity Monitoring Events tab showing /etc/crontab as a 
modified file after the attacker planted the reverse shell persistence 
mechanism. Agent name, timestamp, and file path all captured. Detection 
occurred within seconds of the modification.

## Remediation Confirmed
Terminal output showing the clean crontab after removing attacker 
entries, ls -la /bin/bash confirming SUID bit removed (-rwxr-xr-x, 
no 's'), and SSH returning 'Permission denied (publickey)' when 
attempting password-based login from Kali — confirming the hardening 
worked.
