---
{"dg-publish":true,"permalink":"/cards/active-directory/ntds-database-structure/"}
---

~ [[cards/active-directory/windows active directory\|Active Directory]] | ~ [[cards/red-team/Credential Harvesting\|Credential Harvesting]]
## How it works
---
New Technologies Directory Services (NTDS) is a database containing all Active Directory data, including objects, attributes, credentials, etc. The NTDS.DTS data consists of three tables:

- **Schema table:** Contains types of objects and their relationships
- **Link table:** Contains object's attributes and their values  
- **Data type:** Contains users and groups

**Location:** `C:\Windows\NTDS` (default)

**Encryption:** NTDS.dit file is encrypted to prevent data extraction from target machine

**Access Restriction:** Accessing NTDS.dit file from running machine is disallowed since file is used by Active Directory and is locked

**Decryption Requirement:** Decrypting NTDS file requires system Boot Key stored in `SECURITY` file system to attempt decrypting LSA Isolated credentials