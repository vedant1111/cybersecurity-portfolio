# SOC Incident Report: Credential Compromise & Privilege Escalation

## 1. Incident Timeline
*   **Host:** Windows-VM[cite: 5]
*   **User:** `<username>`[cite: 5]
*   **[10:02]** Multiple failed logon attempts detected, indicating password guessing activity[cite: 5].
*   **[10:04]** Successful logon for the same user, suggesting credentials were compromised[cite: 5].
*   **[10:05]** Special privileges assigned (Event ID 4672), granting privileged access after login[cite: 5].
*   **[10:06]** Privileged group enumeration (Event IDs 4798/4799) identified as post-authentication reconnaissance activity[cite: 5].
*   **[10:08]** Host integrity alert (Rootcheck – Level 7) generated, indicating a system-level anomaly requiring review[cite: 5].


## 2. Threat Hunt & Investigation
*   **Hypothesis:** If credentials were compromised, the attacker may attempt privilege escalation, persistence, or lateral movement[cite: 3].
*   **Hunting Results:** Privileged access and administrative group enumeration were observed post-authentication[cite: 3]. There was no evidence of new user creation, group membership changes, lateral movement, log tampering, or defense evasion[cite: 3].
*   **Conclusion:** The investigation indicates a likely credential compromise followed by privilege reconnaissance activity[cite: 3]. The incident appears contained to a single endpoint[cite: 3].


## 3. Escalation & Response
*   **Severity:** High[cite: 4]
*   **Impact Assessment:** Unauthorized access using valid credentials combined with privileged activity could lead to data exposure, unauthorized system changes, or further compromise if not promptly investigated and contained[cite: 4].
*   **Recommended Actions:** 
    *   Reset credentials for the affected user account[cite: 4].
    *   Review recent privileged account activity[cite: 4].
    *   Monitor the environment for lateral movement attempts[cite: 4].
    *   Validate host integrity and system configuration[cite: 4].
*   **Status:** Escalated for further investigation and monitoring[cite: 4].