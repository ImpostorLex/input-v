---
{"dg-publish":true,"permalink":"/cards/active-directory/lightweight-directory-access-protocol/","tags":["windows/ad"]}
---

[[map-of-contents/active-directory\|active-directory]]
### Introduction 
---
A mechanism that enables accessing and managing directory data such users, computers, group, and any network accessible devices, it is used in any directory service such as Active Directory.

Retrieve all users in the domain:

```
Get-ADUser -Filter *
```

**Distinguished Name (DN)**

`DN = CN=Mayor Malware, OU=Management, DC=wareville, DC=thm`

- **Common Name** specifies the object (user, computer, or any entity)
- **Organizational Unit**
- **Domain Component** each part of a domain name is represented as DC
	- `wareville.thm` = `DC=wareville` and `DC=thm`

## Prerequisites

## Subtopic 1

### Questions and Problems
---
**Is LDAP limited to Active Directory Only?**
No, the protocol can be used with any directory services that organizes information in a hierarchical structure.

**Why LDAP splits domain name into multiple DCs?**
To ensure compatibility with Domain Name System (DNS) as LDAP support **tree-like directory structure**, in this way it is better for organization and scalability.

## Conclusion


