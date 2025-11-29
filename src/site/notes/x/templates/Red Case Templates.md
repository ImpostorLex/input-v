---
{"dg-publish":true,"permalink":"/x/templates/red-case-templates/","title":"INC | YYYYMMDD – [Challenge or Target] – Red Team"}
---

~ [[]] % link to mitre % (if possible)
## Summary
---
**Objective:** What the challenge asked you to accomplish (1–2 lines).  
**Result:** High-level outcome of the exercise.  
**Key Findings:** Important artifacts / credentials / footholds discovered.

## Engagement Context
---
Attack surface, constraints, environment, or scenario details needed to understand the challenge.  
(Example: “Internal AD lab, low-priv foothold, no internet, AV enabled.”)

## Workflow (Steps + Commands + TTP Links)
---
Document your real sequence of actions — tools, thought process, commands, and discoveries.

1. **Action you took**  
   - Command: `{{cmd_here}}`  
   - Output/Observation: …  
   - TTP: [[TECH – T1047 WMI Execution\|TECH – T1047 WMI Execution]]  
2. **Next action**  
   - Command: …  
   - Observation: …  
   - TTP: [[TECH – T1021 SMB Remote Exec\|TECH – T1021 SMB Remote Exec]]  

*(Repeat as needed — messy, chronological, practical.)*

## Observations & Artifacts
---
- Credentials / Tokens  
- Files dropped  
- Process chains  
- Hostnames  
- IPs  
- Output snippets worth saving  
- Any “this was useful” evidence

## Technique Mapping (If TECH note exists)
---
List every technique you used **only once**, even if they appeared multiple times in workflow.

- [[TECH – T1047 / WMI Execution\|TECH – T1047 / WMI Execution]]  
- [[TECH – T1059 / Command Execution\|TECH – T1059 / Command Execution]]  
- [[TECH – T1021 / Remote Services\|TECH – T1021 / Remote Services]]

## Notes for Future Me (Optional)
---
Short, practical bullet points ONLY IF useful.

- “This tool breaks under AMSI; use X next time.”  
- “Alternate attack path existed through LDAP — test next time.”  
- “Time wasted here because I forgot to validate credentials early.”  


