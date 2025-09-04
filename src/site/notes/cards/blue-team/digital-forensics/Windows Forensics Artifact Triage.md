---
{"dg-publish":true,"permalink":"/cards/blue-team/digital-forensics/windows-forensics-artifact-triage/"}
---

[[atlas/blue-team\|blue-team]]
### Introduction
---
Artifacts to prioritize getting when performing the initial triage of a Windows system to quickly perform forensics aside from memory or volatile data.

- [[cards/blue-team/digital-forensics/Order of Volatility\|Order of Volatility]] - it is important to perform MEMORY data acquisation **first.**
## KAPE
---

- [[cards/blue-team/digital-forensics/Common Windows Forensics Artifacts#Kroll Artifact Parser And Extracter (KAPE)\|KAPE]] some notes here.
- It can pretty much do anything we did manually.
- **The advantage of using this is modular**: we can select what we want to analyze first to reduce the overhead and get to forensic analysis immediately then have KAPE analyze while performing forensics.
- **Important:** Always remember scope.
### Triaging
---
Some of the interesting modules and we can also import modules as well:

![Windows Forensics Artifact Triage.png|450](/img/user/cards/blue-team/digital-forensics/Windows%20Forensics%20Artifact%20Triage.png)
At the **target options** there are two powerful collection (Double click to view what is being collected):

- `!BasicCollection`
- `!SANS_Triage`

The above is a great start but these are useful as well:

- `$LogFile` records different transaction logs of file system changes, and can sometimes recover lost data.
- `RecycleBin*`
- `RegistryHives*`
- `$MFT` _master file table_, records all files and directories on the file system including metadata.
- `Amcache` stores information about installed applications.
- `LogFiles` system logs: events and errors.
- `lnk`, and `Prefetch`.
## FTK Imager
---
Do the same thing but with different tool:

**Add Evidence Item -> Physical and then the drive of interest:**

![Windows Forensics Artifact Triage-1.png](/img/user/cards/blue-team/digital-forensics/images/Windows%20Forensics%20Artifact%20Triage-1.png)
Then here are the targets:

![Windows Forensics Artifact Triage-2.png](/img/user/cards/blue-team/digital-forensics/images/Windows%20Forensics%20Artifact%20Triage-2.png)
- Additionally add `UsrClass.dat` as well via Wild Card.
- `*.evtx, *.lnk, *.pf`  and hit all checkboxes.
- jumplist `*.automaticDestination-ms` and custom: `*.customDestination-ms`

Done? create image.