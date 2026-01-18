---
{"dg-publish":true,"permalink":"/cards/red-team/credential-harvesting/"}
---

~ [[atlas/red-team\|red-team]]
### Summary
---

**What you will find:** A high-level overview of Kerberos authentication, ticket flow, delegation concepts, and the most common Kerberos-based attack vectors in Active Directory environments.

**Primary purpose:** Help you identify where you are in the Kerberos authentication chain and select the correct exploitation or detection technique based on your current level of access.

**Key Topics:**

- Kerberos authentication flow (AS-REQ / AS-REP / TGS)
    
- Ticket types (TGT, TGS) and lifetimes
    
- Delegation models (unconstrained, constrained, resource-based)
    
- Credential material exposure via Kerberos
    
- Common Kerberos abuse paths in enterprise AD
    

**Expect to see:**

- Key concepts: TGT/TGS lifecycle, AS-REP/AS-REQ, delegation types
    
- Core techniques: AS-REP Roasting, Kerberoasting
    
- Tools: Rubeus, Kekeo
    
- Case notes / labs: THM Kerberoast Lab
    

---

#### Key Concept 1 — Kerberos (High-Level)

---

Kerberos is the default authentication protocol used in Active Directory to provide secure, ticket-based authentication between users, services, and hosts.

Instead of sending passwords over the network, Kerberos relies on cryptographic tickets issued by the Key Distribution Center (KDC). These tickets prove identity and authorization when accessing network services.

From an attacker perspective, Kerberos becomes a target when ticket material, delegation rights, or service configurations allow credential reuse, offline cracking, impersonation, or privilege escalation.

---

## Knowledge Base

---

%Use this section to anchor core techniques; if a technique grows too large, break it into its own TECH note as shown below.%

### Core Techniques

---

- [[cards/red-team/SAM Database Dumping\|TECH - T1003.002 SAM Database Dumping]]
    
- [[cards/red-team/LSASS Memory Dumping\|TECH - T1003.001 LSASS Memory Dumping]]
    
- [[NTDS.dit Dumping\|TECH - T1003.003 NTDS.dit Dumping]]
    
- [[Harvesting Windows Credential Manager\|TECH - T1555.004 Windows Credential Manager]]
    
- [[LAPS Password Extraction\|TECH - T1003.008 LAPS Password Extraction]]
    

---

## Tools Commonly Used

---

- **PowerShell** — Native AD interaction, enumeration, and post-exploitation
    
- **PsExec** — Remote execution via SMB and service creation
    

---

### Detection & Defense References

---

- **PLAYBOOK** | Hunt — Lateral Movement Indicators
    
- **REF** | MITRE ATT&CK — Detection guidance for TA0008 (Lateral Movement)
    

---

## Case Notes / Labs

---

- **INC** | 20251015 — THM-Blue-Lab-Lateral-Movement
    
- **INC** | 20250920 — HTB — Pivoting Exercise