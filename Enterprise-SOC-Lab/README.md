# Enterprise SOC Lab: Threat Detection Pipeline
An end-to-end Security Operations Center lab demonstrating infrastructure deployment, threat hunting, and incident escalation using Wazuh SIEM and Windows Sysmon.

## ⚙️ SIEM Operations & Telemetry
The Windows 10 endpoint (testagent at 192.168.56.1) was configured with Sysmon to actively log process creation and file access. The Wazuh Agent (v4.7.5) successfully communicated with the Wazuh Manager.
* ![Wazuh Dashboard](https://raw.githubusercontent.com/vedant1111/cybersecurity-portfolio/main/Wazuh_dashboard.png)
* ![Sysmon Active](https://raw.githubusercontent.com/vedant1111/cybersecurity-portfolio/main/Sysmon_active.png)

## 💻 Endpoint Investigation & Detection
Simulated attacks generated actionable security events.
* **Credential Compromise:** Correlated multiple failed logons (Event ID 60122) with a successful workstation logon (Event ID 60118). 
* ![Failed Logons](https://raw.githubusercontent.com/vedant1111/cybersecurity-portfolio/main/Failed_logon_events.png)
* **Privilege Escalation:** Detected special privileges (Event ID 4672) and administrative group enumeration (Event IDs 4798 and 4799).
* ![Privileged Logon](https://raw.githubusercontent.com/vedant1111/cybersecurity-portfolio/main/Event_viewer_Privileged_logon.png)

## 🔍 Incident Correlation & Reporting
Isolated events were correlated by host and time to build a comprehensive timeline. 
* [18:42] - Logon failures detected.
* [18:48] - Privilege access alerts triggered.
* [23:27] - Rootcheck alert (Level 7) identified a potential kernel-level rootkit.
* ![Rootkit Alert](https://raw.githubusercontent.com/vedant1111/cybersecurity-portfolio/main/Rootcheck.png)

**➡️ Read the full Threat Hunt and Escalation findings here:** [Incident Response Report](Incident_Response_Report.md)
