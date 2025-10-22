---
{"dg-publish":true,"permalink":"/cards/active-directory/active-directory/","tags":["#MOC","windows/ad"]}
---

[[cards/active-directory/windows active directory\|windows active directory]]
### Introduction 
---
It has many components but the core function of Active Directory is to provided **Authentication** and **Authorization**.
## Domain Services (AD DS)

- Main component of Active Directory.
- Three main functions:
	- **Directory Service** - Provides methods for storing data in a structured way.
	- **Authentication Service**.
	- **Authorization Service**.
## Core Components

Active Directory organizes resources in a hierarchical logical structure.

![Active Directory.png](/img/user/cards/active-directory/images/Active%20Directory.png)
- **Forest** - a collection of Active Directory Domains
	- Domains in the same forest automatically have a _transitive trust_, a mechanism for users to access resources in another domain - trust is extended beyond a two-domain trust to include other trusted domains.
- **Child** - a domain that shares the same domain name space as the root domain.
- **Trees** - are set of domains that are connected together using the same name space.
- [[cards/active-directory/Kerberos\|Kerberos]] - the default authentication method of Microsoft Windows Domain.
- [[cards/active-directory/Group Policy Objects\|Group Policy Objects]] - includes investigation tips

### Questions and Problems
---
## Conclusion


https://activedirectorypro.com/what-is-active-directory/