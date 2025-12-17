---
{"dg-publish":true,"permalink":"/cards/active-directory/sid-history-concept/"}
---

~ [[cards/active-directory/windows active directory\|Active Directory]]
## How it works
---

**What are SIDs:**

SIDs are used to track security principals (something that can be given permissions and can access resources) and the account's access when connecting to resources.

**Legitimate Use Case:**

SID History enables access for an account to effectively be cloned to another. This is useful when organization performs AD migration, allowing users to retain access to original domain while migrating to new one.

**How It Works:**

When user is created in Active Directory, they get a SID (Security Identifier). That SID is what Windows actually uses to check permissions.

**Problem During AD Migrations:**

- Old domain user: `SID = S-1-5-21-OLD-1234`
- New domain user (after migration): `SID = S-1-5-21-NEW-5678`

All file servers, shares, and applications in old domain still reference old SID in their ACLs.

**Without SID History:**
- User loses access
- Every ACL must be updated (painful and slow)

**Solution: SID History:**

SID History allows you to attach the old SID to the new account.

Now the new account has:
- A new SID (primary identity)
- The old SID stored in SIDHistory

**When user authenticates:**
- Both SIDs are placed into access token
- Old resources still see user as original account

**SID History is Not Limited to Old-Domain SIDs:**

Even though SID History was designed for cross-domain migrations, Active Directory does NOT enforce that rule technically.

So if you have sufficient privileges, you can:
- Add any SID
- From the same domain
- To the SIDHistory attribute of an account

This includes high-privilege group SIDs (requires Domain Admin privileges or equivalent).

**Why SID History Gives Real Privileges:**

When user logs in:

1. Active Directory builds a logon token
2. The token contains:
   - User's primary SID
   - Group SIDs
   - SIDHistory SIDs
3. Windows uses this token to authorize access

**Important detail:**

Windows does NOT distinguish between "real" group membership and SIDHistory SIDs.

If a SID exists in the token, it is trusted. Inject the Enterprise Admin SID and you get the FOREST.