# Remediation SOC Lab – Pyramid of Pain Case Study

## Objective
In this lab I show how I handled a malicious alert using the Pyramid of Pain framework. I started with quick containment at the IOC level by blocking a malicious IP through firewall rules, then moved up the pyramid to disrupt defense evasion by creating a Sigma rule for registry changes at the host artifact level. This highlights a layered SOC approach — detecting, analyzing, and remediating threats across both network and host environments.

---

## Stage 1 – Firewall (IOC Containment)

### Detection
- Alert received for malicious file `sample2.exe` delivered via phishing email.
- Sandbox analysis revealed outbound connection to malicious IP `154.35.10.113`.

![Screenshot Placeholder – Firewall Detection]

### Analysis
- Indicator of Compromise (IOC): Attacker IP address.
- Likely Command & Control (C2) server connection.

### Remediation
- Applied firewall egress rule:
  - **Source IP**: Any  
  - **Destination IP**: 154.35.10.113  
  - **Action**: Deny  
- Effectively stopped all outbound connections to the malicious server.

![Screenshot Placeholder – Firewall Rule]

### Pyramid of Pain Mapping
- **IOC level**: Quick containment, attacker can pivot by changing IP.
- Demonstrates immediate defensive action at the network edge.

### Reflection
- Blocking IPs provides rapid relief but is fragile against attacker adaptation.
- Reinforces the importance of combining IOC‑level containment with higher‑level detection.

---

## Stage 3 – Host Artifacts (Defense Evasion Disruption)

### Detection
- Alert received for malicious file `sample4.exe`.
- Sandbox analysis revealed registry modification:
  - Key: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection`  
  - Name: `DisableRealtimeMonitoring`  
  - Value: `1`

![Screenshot Placeholder – Registry Detection]

### Analysis
- Artifact/TTP: Registry modification to disable Windows Defender real‑time monitoring.
- MITRE ATT&CK Technique: Defense Evasion (TA0005).

### Remediation
- Authored Sigma rule targeting Sysmon registry events:
  ```yaml
  title: Windows Defender Realtime Protection Disabled
  id: thm-lab-002
  status: experimental
  description: Detects registry modification disabling Defender realtime monitoring
  logsource:
    category: registry
    product: windows
  detection:
    selection:
      TargetObject: 'HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection\DisableRealtimeMonitoring'
      Details: 'DWORD (0x00000001)'
    condition: selection
  level: high
  tags:
    - attack.defense_evasion
    - attack.tactic.TA0005
