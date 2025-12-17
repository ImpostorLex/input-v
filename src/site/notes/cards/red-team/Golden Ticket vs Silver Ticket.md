---
{"dg-publish":true,"permalink":"/cards/red-team/golden-ticket-vs-silver-ticket/","tags":["windows/ad"]}
---

~ [[cards/active-directory/windows active directory\|Active Directory]] 
~ [[atlas/red-team\|red-team]]
## Summary
---
### Golden Ticket
---
In normal Kerberos authentication, Golden Tickets bypass steps 1 and 2 where you prove to the DC who you are, creating a forged TGT.

**Requirements:**
- KRBTGT account's password hash to sign a TGT for any user account

**Normal Kerberos Authentication:**

1. **AS-REQ / AS-REP**
   - User proves identity to KDC (Domain Controller)
   - User encrypts timestamp with their own password hash
   - KDC decrypts with stored hash - authentication succeeds

2. **KDC issues TGT**
   - TGT encrypted/signed with KRBTGT account hash
   - User can now request service tickets (TGS-REQ) for any service

**Notice:** Normally you need the user's password hash to get TGT in the first place.

**Golden Ticket Difference:**

- You already have the KRBTGT account hash - the master key of Kerberos in AD
- You can create your own TGT for any user (even if account is disabled or doesn't exist) as long as account is older than 20 minutes
- Because TGT is signed with KRBTGT hash, the KDC trusts it without needing user to authenticate normally

**Advantages:**

- TGTs contain policies and rules for tickets. You can overwrite values pushed by KDC, such as tickets valid only for 10 hours. You can make TGT valid for 10 years granting eternal persistence.
- By default, KRBTGT account's password never changes. Once you have it, you have persistent access by generating TGTs forever.
- Blue team must rotate KRBTGT account password twice, since current and previous passwords are kept valid to ensure accidental rotation does not impact services.
- Rotating KRBTGT account password is a hassle process causing significant services to stop working. Some services are not smart enough to release TGT since timestamp is still valid.
- You can generate Golden Ticket on any machine, even non-domain-joined, making it harder for Blue Team to detect.

### Silver Ticket
---
Skipping all communication from Kerberos authentication process (steps 1-4) and just interacting with the service you want to access directly. Silver Tickets forge Ticket Granting Service (TGS) tickets.

**Key Differences:**

- Generated TGS is signed by machine account of the host you are targeting
- Main difference between Golden and Silver Tickets is scope of privileges acquired:
  - KRBTGT account hash = access to everything
  - Machine account hash = only impersonate users on that specific host
  - Silver Ticket scope limited to whatever service you are targeting on that specific server

**Advantages:**

- No associated TGT since DC was never contacted. Logs only available on targeted server.
- Since permissions determined through SIDs, you can create non-existing user for Silver Ticket as long as ticket has relevant SIDs placing user in host's local administrators group.
- Machine account's password usually rotated every 30 days (not ideal for persistence). You can leverage TGS access to alter host's registry responsible for password rotation of machine account.
- While only having access to single host might seem like downgrade, machine accounts can be used as normal AD accounts, allowing administrative access to host and means to continue enumerating and exploiting AD as you would with AD user account.
