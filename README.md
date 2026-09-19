SOC home lab featuring Wazuh, Sysmon, Suricata, custom detection rules, attack simulation, and threat hunting.
# SOC Detection Engineering & Threat Hunting Home Lab

## Overview

This project is a multi-VM cybersecurity home lab designed to simulate security events, collect endpoint and network telemetry, develop custom detections, and investigate alerts in a SIEM environment.

The lab uses **Wazuh** for centralized security monitoring and detection, **Sysmon** for detailed Windows endpoint telemetry, **Suricata** for network intrusion detection, and **Kali Linux** for controlled attack simulation.

Throughout the project, I configured the environment, integrated multiple telemetry sources, developed and tested custom Wazuh detection rules, simulated adversary activity, and investigated the resulting alerts using Wazuh Threat Hunting.

---

## Architecture

![SOC Home Lab Architecture](soc-homelab-architecture.png)

### Lab Environment

| System | Role | IP Address |
|---|---|---|
| Kali Linux | Attack simulation / network monitoring | 192.168.139.129 |
| Windows 11 | Monitored endpoint | 192.168.139.128 |
| Ubuntu Server | Wazuh SIEM / Manager | 192.168.139.131 |

The lab operates on an isolated VMware network (`192.168.139.0/24`). Kali Linux also uses a separate NAT interface when Internet access is required.

---

## Technologies

- Wazuh
- Sysmon
- Suricata IDS
- Kali Linux
- Windows 11
- Ubuntu Server
- VMware Workstation
- PowerShell
- Nmap
- Windows Event Logging
- Microsoft Defender
- File Integrity Monitoring

---

## Detection Engineering

I developed and tested 11 custom Wazuh detection rules to identify endpoint and network security activity.

| Rule ID | Detection | Severity |
|---|---|---:|
| 100100 | Important Windows file modification | 12 |
| 100101 | `whoami.exe` execution / discovery activity | 5 |
| 100102 | PowerShell lab test activity | 7 |
| 100103 | Kali-originated network connection | 7 |
| 100104 | Possible network scan from Kali Linux | 10 |
| 100105 | Repeated failed logons / possible brute-force activity | 12 |
| 100106 | New Windows user account created | 10 |
| 100107 | User added to local Administrators group | 12 |
| 100108 | Microsoft Defender malware or PUP detection | 14 |
| 100109 | Encoded PowerShell / possible obfuscated execution | 14 |
| 100110 | Scheduled task creation / possible persistence | 12 |

These detections use telemetry from Sysmon, Windows Security events, Microsoft Defender, File Integrity Monitoring, and network activity.

---

## Endpoint Monitoring

Sysmon was deployed on the Windows 11 endpoint to provide detailed telemetry including:

- Process creation
- Command-line execution
- Process relationships
- Network activity
- File activity
- Cryptographic hashes

Windows Security logs and Microsoft Defender telemetry were also forwarded to Wazuh for centralized analysis.

File Integrity Monitoring was configured for security-sensitive files to detect modifications in real time.

---

## Network Monitoring

Suricata was deployed as the network IDS component of the lab.

Suricata telemetry was written to `eve.json` and forwarded through the Wazuh agent for centralized monitoring and analysis.

Network testing from Kali Linux was used to generate controlled traffic and validate network-oriented detections.

---

## Attack Simulation

Controlled security events were generated inside the isolated lab to validate detection coverage.

Examples include:

- Network reconnaissance and Nmap scanning
- Repeated authentication failures
- PowerShell execution
- Encoded PowerShell execution
- Scheduled-task creation
- Account creation
- Privileged-group modification
- File modification
- Network connections from the Kali attack system
- Discovery activity

The purpose of these simulations was not exploitation of external systems, but validation of logging, detection, alerting, and investigation workflows.

---

## Threat Hunting & Investigation

Alerts were investigated using the Wazuh Threat Hunting interface.

Rather than relying only on alert names, I examined underlying event fields such as:

- Command lines
- Process names and IDs
- Parent processes
- User accounts
- Source and destination IP addresses
- Destination ports
- Event IDs
- File hashes
- Alert severity
- Detection rule IDs

This allowed activity to be traced from the original simulated action through endpoint or network telemetry and into the corresponding Wazuh alert.

---

## Example Detection Chain

One test involved executing an encoded PowerShell command on the Windows endpoint.

The activity produced Sysmon process-creation telemetry containing the PowerShell command line. The event was forwarded to Wazuh, where a custom rule identified the `-EncodedCommand` behavior and generated a high-severity alert.

This demonstrated the complete detection pipeline:

`Attack Simulation → Endpoint Telemetry → Wazuh Agent → Wazuh Manager → Custom Detection → Threat Hunting Investigation`

---

## Skills Demonstrated

- SIEM deployment and administration
- Detection engineering
- Custom Wazuh rule development
- Windows endpoint monitoring
- Sysmon configuration and analysis
- Network intrusion detection
- Suricata integration
- Log collection and analysis
- PowerShell telemetry analysis
- Threat hunting
- Alert triage
- Event correlation
- File integrity monitoring
- Attack simulation
- Security troubleshooting
- Virtualized network design

---

## Project Status

The core lab implementation and detection testing are complete.

Current work focuses on documenting investigations, mapping detections to MITRE ATT&CK, and developing portfolio-quality incident analysis.

---

## Disclaimer

All attack simulations and security testing documented in this project were performed in my own isolated lab environment for educational purposes.
