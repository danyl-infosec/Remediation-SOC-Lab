# Remediation SOC Lab – Pyramid of Pain Case Study

## Objective
In this lab I walk through how I handled a malicious alert using the Pyramid of Pain framework. I started with quick containment at the IOC level by blocking a malicious IP through firewall rules, then moved up the pyramid to disrupt defense evasion by creating a Sigma rule for registry changes at the host artifact level. This highlights a layered SOC approach — detecting, analyzing, and remediating threats across both network and host environments.

---

## Firewall (IOC Containment)

### Detection
I was alerted to a malicious file delivered in a phishing email. After sandboxing `sample2.exe`, I observed that the process attempted to connect to a webpage hosting a malicious IP (`154.35.10.113`).

![Lab 3 Screenshot 1](https://raw.githubusercontent.com/danyl-infosec/Remediation-SOC-Lab/refs/heads/main/Lab%203%20Screenshot%201.png)

### Analysis
- IOC identified: outbound connection to `154.35.10.113`.  
- Likely Command & Control (C2) server activity.  

![Lab 3 Screenshot 3](https://raw.githubusercontent.com/danyl-infosec/Remediation-SOC-Lab/refs/heads/main/Lab%203%20Screenshot%20%233.png)

### Remediation
I went to the firewall manager and created an **egress rule** to block outgoing connections to the malicious IP. Since the compromised machine was inside the network, I set:
- **Source IP**: Any  
- **Destination IP**: 154.35.10.113  
- **Action**: Deny  

This effectively stopped all outbound traffic to the attacker’s C2 server.

![Lab 3 Screenshot 2](https://raw.githubusercontent.com/danyl-infosec/Remediation-SOC-Lab/refs/heads/main/Lab%203%20Screenshot%202.png)

### Pyramid of Pain Mapping
- **IOC level**: Quick containment, but the attacker can easily pivot by changing IP using techniques like fast flux.  

### Reflection
Blocking IPs provides rapid relief but is fragile against attacker adaptation. It reinforces the need to escalate detection to higher levels of the Pyramid of Pain.

---

## Host Artifacts (Defense Evasion Disruption)

### Detection
After the initial IOC containment, the attacker pivoted to evade simple defenses like domain/IP blocking. I obtained another malicious file (`sample4.exe`) for analysis and ran it in a sandbox to observe its behavior. During execution, I noticed suspicious registry activity. The malware attempted to disable Windows Defender’s real‑time monitoring by modifying:

- **Key**: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection`  
- **Name**: `DisableRealtimeMonitoring`  
- **Value**: `1`

![Lab 3 Screenshot 5](https://raw.githubusercontent.com/danyl-infosec/Remediation-SOC-Lab/refs/heads/main/Lab%203%20Screenshot%20%235.png)

### Analysis
- Artifact/TTP: Registry modification to disable Defender protections.  
- MITRE ATT&CK Technique: Defense Evasion (TA0005).  

### Remediation
To counter this, I wrote a Sigma rule targeting Sysmon registry events. The rule detects attempts to disable Defender’s real‑time protection:

- **Registry Key**: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection`  
- **Registry Name**: `DisableRealtimeMonitoring`  
- **Value**: `1`  
- **ATT&CK ID**: Defense Evasion (TA0005)  

![Lab 3 Screenshot 8](https://raw.githubusercontent.com/danyl-infosec/Remediation-SOC-Lab/refs/heads/main/Lab%203%20Screenshot%208.png)

### Pyramid of Pain Mapping
- **Artifact/TTP level**: Much harder for attackers to change, disrupts their defense evasion strategy.  
- Demonstrates proactive detection and long‑term resilience.  

### Reflection
By moving beyond IOC blocking to artifact/TTP detection, I created lasting pain for the attacker. This shows the ability to translate sandbox observations into actionable Sigma rules tied to MITRE ATT&CK.

---

## Closing Note
This lab demonstrates layered defense:
- **Stage 1**: Immediate containment via firewall egress block (IOC).  
- **Stage 3**: Strategic disruption via Sigma rule for registry defense evasion (Artifact/TTP).  

Together, they show breadth (network + host) and depth (IOC + TTP) — a recruiter‑ready case study that highlights practical SOC remediation skills.
