---
{"dg-publish":true,"permalink":"/cards/active-directory/dc-sync/"}
---

~ [[cards/active-directory/windows active directory\|Active Directory]]
### Summary
---
It is not recommended to have a single DC for multiple domains as it would delay authentication services. So multiple DCs exists. If you want to authenticate using the same credentials in two different offices.

You do **domain replication**, each domain controller runs a process called the Knowledge Consistency Checker (KCC), It generates a replication topology for the AD forest and automatically connects to other domain controllers through Remote Procedure Call (RPC) to synchronise information.

 This includes updated information such as the user's new password and new objects such as when a new user is created.

This process of replication is called **DC Synchronisation**. Those who are part of the Domain Admins group can also do DC synchronisation for legitimate purposes such as creating a new domain controller.

