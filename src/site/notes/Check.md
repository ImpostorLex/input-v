---
{"dg-publish":true,"permalink":"/check/"}
---



This checklist is the simplified, practical version of what analysts should keep in mind during alert triage and investigation.

# 1. Determine Normal vs Abnormal
------------------------------------------------------------
Ask the following:
- Is this normal behavior for the user?
- Is this normal for the host or application?
- Has this been seen before?
- Is the time of activity normal?
- Is the source IP, domain, or device expected?

If anything is unusual, mark for deeper investigation.

------------------------------------------------------------
# 2. Timeline Context (Before and After)
------------------------------------------------------------
Always determine what happened:
- Before the alert (possible cause)
- During the alert (the event itself)
- After the alert (possible impact)

Look 5–10 minutes on both sides of the alert timestamp.

------------------------------------------------------------
# 3. Identify Rare or New Activity
------------------------------------------------------------
Check for the following indicators:
- New process or command never seen on the host
- New source IP or login location
- New parent-child process relationship
- New destination domain or external IP
- New login type (ex: service account logging in interactively)

Anything rare or new should be considered suspicious.

------------------------------------------------------------
# 4. Match Behavior to Common Attack Patterns
------------------------------------------------------------
Check if the activity fits common categories:
- Execution (scripts, PowerShell, unusual binaries)
- Persistence (scheduled tasks, services, registry changes)
- Privilege escalation (token use, admin behavior)
- Lateral movement (RDP, SMB, WinRM)
- Credential access (LSASS access, password dumping)
- Exfiltration (large outbound traffic, unusual destinations)

If it fits a known attack pattern -> escalate.

------------------------------------------------------------
# 5. Validate Identity and Access Use
------------------------------------------------------------
For any alert involving authentication:
- Is the user supposed to access this system?
- Are there failed logins before success?
- Is the login type appropriate (service account shouldn't be interactive)?
- Is MFA used or bypassed?
- Are there logins from new geographic or network locations?

If identity looks compromised -> escalate.

------------------------------------------------------------
# 6. Determine the Responsible Process or Actor
------------------------------------------------------------
Identify what actually performed the action:
- Which process?
- Which user?
- Which host?
- Which network connection?

If process tree is unavailable, rely on:
- Command line logs
- Network telemetry
- Authentication logs
- File events
- Script or PowerShell logs

------------------------------------------------------------
# 7. Assess Blast Radius
------------------------------------------------------------
Ask:
- Did this activity happen on other hosts?
- Does the same IOC appear elsewhere (hash, IP, domain)?
- Are there multiple affected accounts?
- Are other alerts linked?

If the activity appears across more systems -> escalated incident.

------------------------------------------------------------
# 8. Final Decision: Benign, Suspicious, or Malicious
------------------------------------------------------------
Use this logic:
- Benign -> Clearly normal and explainable
- Suspicious -> Needs containment or deeper investigation
- Malicious -> Confirmed compromise behavior

Document reasoning for whichever category is selected.

------------------------------------------------------------
# 9. Tier 1 -> Tier 2 Escalation Criteria
------------------------------------------------------------
Escalate if any of the following apply:
- Activity is new, rare, or unexplained
- There is evidence of credential misuse
- There is lateral movement behavior
- There are repeated correlated alerts
- There is potential data access or exfiltration
- Any persistence mechanism is detected
- EDR/SIEM correlation suggests ongoing activity

Tier 1 analysts should provide:
- The alert summary
- Relevant logs (auth, process, network)
- Timeline reconstruction
- Observed anomalies
- Any matched IOCs

------------------------------------------------------------
# 10. Tier 2 Expected Actions
------------------------------------------------------------
Tier 2 should:
- Deep-dive into host and network logs
- Pivot across entities (user, host, IP, hash)
- Build full timeline
- Validate persistence or escalation attempts
- Identify attacker goals or impact
- Recommend containment or response actions

------------------------------------------------------------
# Summary (Fast Recall)
------------------------------------------------------------
1. Is it normal?
2. What happened before and after?
3. Is anything rare or new?
4. Does it fit an attack pattern?
5. Is identity being misused?
6. What process/user/system did it?
7. How big is the blast radius?
8. Decide: benign, suspicious, malicious
9. Escalate if the activity is abnormal or harmful
