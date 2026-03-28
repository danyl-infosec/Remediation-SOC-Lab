# Remediation SOC Lab – Pyramid of Pain Case Study

## Objective
In this lab I show how I handled a malicious alert using the Pyramid of Pain framework. I started with quick containment at the IOC level by blocking a malicious IP through firewall rules, then moved up the pyramid to disrupt defense evasion by creating a Sigma rule for registry changes at the host artifact level. This highlights a layered SOC approach — detecting, analyzing, and remediating threats across both network and host environments.

---

## Firewall (IOC Containment)

### Detection
- Alert received for malicious file `sample2.exe` delivered via phishing email.
- Sandbox analysis revealed outbound connection to malicious IP `154.35.10.113`.

![Lab 3 Screenshot 1](https://raw.githubusercontent.com/danyl-infosec/Remediation-SOC-Lab/refs/heads/main/Lab%203%20Screenshot%201.png)


![Lab 3 Screenshot 3](https://raw.githubusercontent.com/danyl-infosec/Remediation-SOC-Lab/refs/heads/main/Lab%203%20Screenshot%20%233.png)


### Analysis
- Indicator of Compromise (IOC): Attacker IP address.
- Likely Command & Control (C2) server connection.

### Remediation
- Applied firewall egress rule:
  - **Source IP**: Any  
  - **Destination IP**: 154.35.10.113  
  - **Action**: Deny  
- Effectively stopped all outbound connections to the malicious server.

![Lab 3 Screenshot 2](https://raw.githubusercontent.com/danyl-infosec/Remediation-SOC-Lab/refs/heads/main/Lab%203%20Screenshot%202.png)


### Pyramid of Pain Mapping
- **IOC level**: Quick containment, attacker can pivot by changing IP.
- Demonstrates immediate defensive action at the network edge.

### Reflection
- Blocking IPs provides rapid relief but is fragile against attacker adaptation.
- Reinforces the importance of combining IOC‑level containment with higher‑level detection.

---

## Host Artifacts (Defense Evasion Disruption)

### Detection
- Alert received for malicious file `sample4.exe`.
- Sandbox analysis revealed registry modification:
  - Key: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection`  
  - Name: `DisableRealtimeMonitoring`  
  - Value: `1`

![Lab 3 Screenshot 5](https://raw.githubusercontent.com/danyl-infosec/Remediation-SOC-Lab/refs/heads/main/Lab%203%20Screenshot%20%235.png)


### Analysis
- Artifact/TTP: Registry modification to disable Windows Defender real‑time monitoring.
- MITRE ATT&CK Technique: Defense Evasion (TA0005).

### Remediation
- Authored Sigma rule targeting Sysmon registry events:
- **Registry Key**: HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection
- **Registry Name**: Disable Realtime Monitoring
- **Value**: 1
- **Att&ck ID**: Defence Evasion TA0005
  
![Lab 3 Screenshot 8](https://raw.githubusercontent.com/danyl-infosec/Remediation-SOC-Lab/refs/heads/main/Lab%203%20Screenshot%208.png)

