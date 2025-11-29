---
{"dg-publish":true,"permalink":"/cards/active-directory/kerberos-delegation/","tags":["windows/ad"]}
---

~ [[cards/red-team/Exploiting Kerberos Delegation\|Exploiting Kerberos Delegation]] 
~ [[cards/active-directory/windows active directory\|Active Directory]]
### Summary
---
Kerberos Delegation allows a service (like a web server) to authenticate to another service (like SQL) **using the original user’s identity**, so the backend system can enforce the user's permissions directly.

## How it works
---
When a user logs into the web app, the web server needs to fetch data from the SQL database. But here is the problem:

**Without delegation**, the SQL server **cannot know which user is making the request**.

The web server must authenticate to SQL using **its own account** (a service account). This service account must be granted permissions to all necessary data.

But this is dangerous:

- The service account can access EVERYTHING the app needs

- Users may only be allowed to access SOME data

- The database cannot enforce **per-user** permissions

- The web server "impersonates everyone as itself"

With **Kerberos Delegation** allows the web server to say:

"Give me a ticket to access the SQL, but **as the user** who logged into the web app."

- Of course, it needs to use Kerberos authentication.
- SQL can now ask "Is **user** allowed to read this table?"

## Constrained vs Unconstrained
---
These are the two types of Kerberos Delegation.
### Unconstrained
---
The original implementation of Kerberos Delegation uses Unconstrained Delegation which is the least secure method. Essentially **Unconstrained provides no limits to the delegation**.

A computer or service with **unconstrained delegation** is allowed to impersonate **any user** to **any service**.

#### How it works, why this is dangerous, and how it is exploited
---
When **any user** logs into the server:  

- Their **TGT** (the most powerful Kerberos ticket) is placed in memory  
- So the server can reuse it for impersonation later

If an attacker gets control of a machine with unconstrained delegation:

- They can **steal any TGT** from memory
- Including TGTs from privileged accounts (Domain Admins, service accounts, etc.)
- Then impersonate that user to anything in the domain

Real world exploitation:

1. Compromise a machine with unconstrained delegation
2. Force a Domain Admin to authenticate to it.
3. Steal the Domain Admin’s **TGT**
4. Become Domain Admin

### Constrained
---
The service can **only impersonate users for specific services** that an admin selects.

Example:

A web server can be configured to impersonate users **only to this**:

```C
MSSQL/DBServer.domain.com
```

#### But it can still be abused
---

Suppose you compromise an account:

```
serviceAccountA
```

And in AD this account is configured with constrained delegation like:

```C
AllowedToDelegateTo:
   MSSQL/SQL01.domain.com
```

- serviceAccountA can impersonate users
- BUT **only to the SQL service on SQL01**

By knowing the plaintext password or even just the NTLM hash of this account, we could generate a TGT for this account, then use the TGT to execute a ticket-granting server (TGS) request for any non-sensitive user account in order to access the service as that user. Imagine impersonating an account with access to a sensitive database.
### Resource-Based Constrained Delegation
---
Is the opposite of Constrained Delegation, it fips the script:

> Instead of saying **"this account can delegate to SQL" we say "SQL will accept delegation from these accounts."**

- Web app server -> needs to access SQL
- SQL server admin wants to control who can impersonate users to SQL
- They add the web server account to:

```C
msDS-AllowedToActOnBehalfOfOtherIdentity
```

on the **SQL server’s computer object**

This attribute says:

> "These accounts are allowed to impersonate users to me." in other words "I trust these accounts to pretend to be any user when they authenticate to me."

#### Example
---
**You have a SQL server: SQL01**

On its AD object, the attribute:

```C
msDS-AllowedToActOnBehalfOfOtherIdentity
```

contains the web server account:

```C
WEBAPP01$
```

This means, when `WEBAPP01$` connects to `SQL01`,

SQL01 will allow WEBAPP01$ to say:

- "I am Alice" -> SQL01 accepts it
    
- "I am Bob" -> SQL01 accepts it
    
- "I am Domain Admin" -> SQL01 accepts it
    
Because SQL01 trusts WEBAPP01$ to impersonate users _on its behalf_.

#### Why this is dangerous
---
If an attacker have permission to configure RBCD for a service they can add their own machine account such as `FAKE01$` into that attribute then **they can impersonate ANY user to SQL01**.

All because SQL01 now trusts `FAKE01$` to pretend to be anyone.
