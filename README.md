# \# SOC Home Lab — Attack, Detect, Respond

# 

# \## Overview

# A fully functional three-VM cybersecurity home lab built from scratch using VMware Workstation Pro. This lab simulates a real-world cyberattack scenario — from initial reconnaissance through privilege escalation and persistence — and demonstrates full SOC analyst detection and incident response using a live SIEM.

# 

# \## Lab Architecture

# \- \*\*Wazuh SIEM\*\* — Ubuntu Server 22.04 running Wazuh manager, indexer, and dashboard. Central log collection and real-time alerting.

# \- \*\*Ubuntu Target\*\* — Ubuntu Desktop 22.04 acting as a victim workstation with Wazuh agent installed.

# \- \*\*Kali Linux\*\* — Attacker machine running industry-standard offensive security tools.

# 

# All three machines run on an isolated VMware host-only network with no connection to the real internet during attack exercises.

# 

# \## Attack Scenario — Privilege Escalation Kill Chain

# 1\. \*\*Reconnaissance\*\* — Nmap SYN scan to identify live hosts and open ports

# 2\. \*\*Brute Force\*\* — Hydra SSH credential attack against the target

# 3\. \*\*Initial Access\*\* — SSH login using cracked credentials

# 4\. \*\*Privilege Escalation\*\* — SUID bash exploitation (euid=0 from standard user)

# 5\. \*\*Persistence\*\* — Reverse shell cron job planted in /etc/crontab

# 

# \## Detection — Wazuh SIEM Alerts Fired

# | Rule | Description | Severity |

# |------|-------------|----------|

# | 5760 | SSH authentication failures (brute force) | Level 5 |

# | 5715 | Successful SSH login after failures | Level 3 |

# | 5402 | Sudo to ROOT executed | Level 3 |

# | FIM syscheck | /etc/crontab modified | Level 7 |

# 

# \## Incident Response — NIST Framework

# \- \*\*Contain\*\* — Removed malicious cron job entries

# \- \*\*Eradicate\*\* — Removed SUID bit from /bin/bash, audited all SUID binaries

# \- \*\*Harden\*\* — Disabled SSH password authentication, installed fail2ban, reset compromised credentials

# \- \*\*Detect\*\* — Wrote custom Wazuh detection rules 100002, 100003, 100004 at critical severity

# 

# \## Custom Wazuh Detection Rules Written

# \- Rule 100002 — SSH brute force detection (Level 15 Critical)

# \- Rule 100003 — SUID bash privilege escalation detection (Level 15 Critical)

# \- Rule 100004 — Crontab modification detection (Level 15 Critical)

# 

# \## Tools Used

# \- VMware Workstation Pro

# \- Wazuh SIEM (Manager + Indexer + Dashboard)

# \- Nmap

# \- Hydra

# \- Kali Linux

# \- auditd

# \- fail2ban

# 

# \## Skills Demonstrated

# \- Virtual machine deployment and network configuration

# \- Linux system administration

# \- SIEM deployment, configuration, and custom rule authoring

# \- File Integrity Monitoring

# \- SSH hardening and access control

# \- Offensive security — network scanning, credential brute forcing, privilege escalation

# \- Incident response following NIST framework

# \- Threat detection and log analysis

