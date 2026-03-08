---
{"dg-publish":true,"permalink":"/cards/red-team/mmc-rsat-enumeration/"}
---

~ [[cards/red-team/Enumerating Active Directory (Authenticated Enumeration)\|Enumerating Active Directory (Authenticated Enumeration)]]
## Summary
---
**What it is:** Using Microsoft Management Console (MMC) with Remote Server Administration Tools (RSAT) Active Directory snap-ins to graphically enumerate AD users, groups, computers, and organizational units from non-domain-joined or domain-joined Windows machines with injected credentials.

**Scope:** This note covers installing RSAT features, launching MMC with injected credentials via runas, adding and configuring AD snap-ins, and enumerating users, groups, OUs, and computer objects through the graphical interface.

**Prerequisites:**

- [[Microsoft Management Console (MMC)\|Microsoft Management Console (MMC)]]
- [[cards/red-team/Credential Injection via Runas\|TECH - T1078.002 - Active Directory Credential Injection via Runas]]
- [[Active Directory Organizational Units\|Active Directory Organizational Units]]
- Valid AD credentials
- Windows machine with RSAT AD tools installed (or ability to install)
- Network access to Domain Controller

**Risks & Limitations:**

- Requires RDP or physical access to Windows machine
- RSAT installation may require local administrator privileges
- Cannot gather AD-wide properties or attributes efficiently (GUI limitation)
- Enumeration is manual and time-consuming for large environments
- Non-domain-joined machines require runas credential injection
- Changes to AD objects require appropriate permissions
- Limited to visual exploration (no scripting or bulk queries)
## How It Works (2-4 lines)
---
Microsoft Management Console (MMC) provides a graphical framework for Windows administrative tools including Remote Server Administration Tools (RSAT) Active Directory snap-ins that allow administrators to manage users, groups, computers, and organizational units. Administrators use MMC with AD snap-ins for daily AD management tasks including user creation, group membership changes, and permission reviews. On non-domain-joined machines, attackers inject AD credentials using runas to launch MMC, then add AD snap-ins (Active Directory Users and Computers, Sites and Services, Domains and Trusts) and point them to the target domain, enabling visual enumeration of AD structure, user attributes, group memberships, and computer objects to identify privilege escalation paths and high-value targets.

## Steps (Hands-On)
---

**Legend:**

- `{DOMAIN}` = za.tryhackme.com
- `{AD_USERNAME}` = Injected AD username
- `{JUMP_HOST}` = THMJMP1 (simulated foothold)
- `{OU_NAME}` = Organizational Unit name
- `{USER_NAME}` = Target user for enumeration
- `{GROUP_NAME}` = Target group for enumeration
- `{COMPUTER_NAME}` = Target computer for enumeration

**Context:** You have obtained AD credentials and need to enumerate Active Directory structure visually. THMJMP1 simulates a foothold that you have achieved in the environment.

---

### 1. Install RSAT AD Tools (If Not Present)

**Action:**

- On Windows 10/11:
  1. Press **Start**
  2. Search **"Apps & Features"** and press Enter
  3. Click **Manage Optional Features**
  4. Click **Add a feature**
  5. Search for **"RSAT"**
  6. Select **"RSAT: Active Directory Domain Services and Lightweight Directory Tools"**
  7. Click **Install**

**Observable:**

- `explorer.exe` or Settings app process
- Windows Update service activity
- Network connections to Microsoft update servers
- RSAT package download and installation
- Event ID 2 (Package installation started) in Microsoft-Windows-AppXDeployment-Server
- Event ID 4 (Package installation completed)
- Files installed to `C:\Windows\System32\` and related directories
- Registry changes under `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing`
- PowerShell modules added to `C:\Windows\System32\WindowsPowerShell\v1.0\Modules\ActiveDirectory\`

**Note:** Domain-joined machines typically have RSAT pre-installed. This step only needed for non-domain-joined machines.

---

### 2. Launch MMC with Injected Credentials

**Action:**

- If non-domain-joined: Use runas command prompt from previous credential injection
- If domain-joined: Can run MMC normally

**Non-domain-joined:**
```C
mmc
```

(Execute from runas command prompt with injected AD credentials)

**Domain-joined:**

- Press **Start** -> Search **"run"** -> Type `mmc` -> Enter

**Observable:**

**Non-domain-joined:**
- `mmc.exe` spawned from runas command prompt
- Process tree: original cmd -> runas.exe -> cmd (with creds) -> mmc.exe
- Event ID 4688 (Process creation) - mmc.exe
- MMC network connections will use injected AD credentials

**Domain-joined:**
- `mmc.exe` launched directly
- Event ID 4688 (Process creation)
- MMC uses current logged-in user credentials

![Enumerating Active Directory-8.jpg](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-8.jpg)

---

### 3. Add Active Directory Snap-ins

**Action:**

- In MMC console:
  1. Click **File** -> **Add/Remove Snap-in**
  2. Select and **Add** all three Active Directory Snap-ins:
     - **Active Directory Users and Computers**
     - **Active Directory Sites and Services**
     - **Active Directory Domains and Trusts**
  3. Click **OK**
  4. Click through any errors and warnings

**Observable:**

- MMC loading snap-in DLL files
- Registry reads under `HKLM\SOFTWARE\Microsoft\MMC\SnapIns`
- LDAP queries to Domain Controller on port 389/636 (initial connection attempts)
- Event ID 4662 (Operation performed on AD object) on DC if detailed auditing enabled
- Snap-ins loaded into MMC console tree
- Potential error dialogs if not pointing to correct domain yet

---

### 4. Configure Active Directory Domains and Trusts

**Action:**

- Right-click on **Active Directory Domains and Trusts**
- Select **Change Forest**
- Enter `{DOMAIN}` as the **Root domain**
- Click **OK**

**Observable:**

- LDAP connection to `{DOMAIN}` Domain Controller
- DNS query to resolve `{DOMAIN}`
- Event ID 4624 (Logon) on DC:
  - Logon Type 3 (Network)
  - Account: `{AD_USERNAME}@{DOMAIN}`
  - Source: Workstation IP
- Event ID 4662 (Operation performed on AD object) - forest configuration query
- Snap-in now connected to target domain forest
- Domain trust information retrieved

---

### 5. Configure Active Directory Sites and Services

**Action:**

- Right-click on **Active Directory Sites and Services**
- Select **Change Forest**
- Enter `{DOMAIN}` as the **Root domain**
- Click **OK**

**Observable:**

- LDAP queries to DC retrieving site and replication topology
- Event ID 4662 (Operation performed on AD object)
- Sites, subnets, and site links displayed
- Replication configuration visible
- Domain controller placement information retrieved

---

### 6. Configure Active Directory Users and Computers

**Action:**

- Right-click on **Active Directory Users and Computers**
- Select **Change Domain**
- Enter `{DOMAIN}` as the **Domain**
- Click **OK**
- Right-click on **Active Directory Users and Computers** in left pane
- Click **View** -> **Advanced Features**

**Observable:**

- LDAP queries to DC retrieving domain structure
- Event ID 4662 (Operation performed on AD object) - domain query
- Domain tree structure populated in MMC
- Advanced Features enabled (shows hidden containers like AdminSDHolder)
- Event ID 4624 (Logon) on DC if new connection established
- Full AD hierarchy now visible

![Enumerating Active Directory-7.jpg|450](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-7.jpg)

**Note:** 
- Domain-joined machines with RSAT pre-installed can run MMC normally without runas
- If domain-joined but logged in with local account or different domain account without rights, runas /netonly can force MMC to use other AD credentials for network authentication

---

### 7. Enumerate Organizational Unit Structure

**Action:**

- Expand the AD snap-in
- Expand `{DOMAIN}` domain
- View Initial Organizational Unit (OU) structure:
  - Builtin
  - Computers
  - Domain Controllers
  - ForeignSecurityPrincipals
  - Managed Service Accounts
  - People
  - Servers
  - System
  - Users
  - Workstations

![Enumerating Active Directory-3.jpg|400](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-3.jpg)

**Observable:**

- LDAP queries retrieving OU structure
- Event ID 4662 (Operation performed on AD object) - OU enumeration
- Multiple LDAP searches for organizational units
- Distinguished Names (DNs) of OUs retrieved
- OU tree structure displayed in MMC

---

### 8. Enumerate Users in Organizational Units

**Action:**

- Navigate to **People** directory
- Explore departmental OUs (IT, Marketing, Sales, etc.)
- Click on specific OU to view users

![Enumerating Active Directory-4.jpg|450](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-4.jpg)

**Observable:**

- LDAP queries to DC for user objects in selected OU
- Event ID 4662 (Operation performed on AD object) - user enumeration
- User list displayed with:
  - Display Name
  - User Logon Name
  - Description
  - Office location
- Multiple user objects retrieved per OU

---

### 9. Enumerate User Properties and Group Membership

**Action:**

- Double-click on specific user (e.g., `{USER_NAME}`)
- Review user properties tabs:
  - General (name, description, office, phone)
  - Account (logon name, account options, expiration)
  - Member Of (group memberships)
  - Security (permissions on user object)

**Observable:**

- LDAP query retrieving full user object attributes
- Event ID 4662 (Operation performed on AD object) - user attribute read
- User properties dialog displays:
  - Account settings
  - Group memberships
  - Organizational info
  - Security permissions
  - Password settings
  - Logon hours
  - Profile path
  - Home directory

![Enumerating Active Directory-5.jpg](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-5.jpg)

**Note:** Group membership visible here allows identifying privileged users and potential targets.

---

### 10. Enumerate Domain Computers

**Action:**

- Navigate to **Servers** or **Workstations** OU
- View list of domain-joined machines

![Enumerating Active Directory-6.jpg|450](/img/user/cards/red-team/images/Enumerating%20Active%20Directory-6.jpg)

**Observable:**

- LDAP queries for computer objects
- Event ID 4662 (Operation performed on AD object) - computer enumeration
- Computer list displayed with:
  - Computer Name
  - DNS Name
  - Operating System
  - Description
  - Last Logon timestamp
- Computer objects retrieved from domain

**Note:** This reveals domain-joined machines that may be targets for lateral movement or privilege escalation.

---

### 11. Modify AD Objects (If Permissions Allow)

**Action:**

- Right-click on user or group object
- Modify properties:
  - Change user password
  - Add user to group
  - Modify group membership
  - Update object attributes

**Observable:**

- LDAP modify operations to DC
- Event ID 5136 (Directory Service object modified) on DC:
  - Object: Modified user/group DN
  - AttributeLDAPDisplayName: Modified attribute
  - AttributeValue: New value
  - Subject: `{AD_USERNAME}`
- Event ID 4662 (Operation performed on AD object) - write operation
- Event ID 4738 (User account changed) if modifying user
- Event ID 4728 (Member added to security-enabled global group) if adding to group
- Changes reflected immediately in AD

**Note:** Modifications require appropriate permissions. Low-privileged accounts typically have read-only access.

---

## Blue - Detection & Response
---

**Indicators of Compromise:**

- Event ID 4624 (Logon Type 3) from non-standard workstations or IPs to Domain Controllers
- Event ID 4662 (Operation performed on AD object) with rapid succession of OU, user, and computer enumeration from single source
- Event ID 4688 (Process creation) - mmc.exe launched from runas command prompt on non-domain-joined machines
- LDAP queries from workstations not typically performing AD management
- Multiple Event ID 4662 showing comprehensive AD object enumeration (users, groups, computers, OUs)
- MMC.exe network connections to Domain Controller port 389/636 from unexpected sources
- Enumeration of privileged groups (Domain Admins, Enterprise Admins) by non-admin accounts
- RSAT installation on non-standard workstations
- Event ID 2 and 4 (Package installation) for RSAT components on user workstations
- Advanced Features enabled in AD Users and Computers (EventID 4662 accessing hidden containers like AdminSDHolder)
- Comprehensive user attribute queries (Member Of, Security permissions) from low-privileged accounts
- Computer object enumeration across multiple OUs in short timeframe
- GUI-based AD enumeration during non-business hours
- Single account enumerating hundreds of users, groups, and computers within minutes