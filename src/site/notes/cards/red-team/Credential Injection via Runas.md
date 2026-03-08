---
{"dg-publish":true,"permalink":"/cards/red-team/credential-injection-via-runas/"}
---

~ [[atlas/red-team\|red-team]]
## Summary
---
**What it is:** Injecting valid Active Directory credentials into memory on a non-domain-joined Windows machine using runas.exe with /netonly flag to enable network authentication to AD resources without local domain authentication.

**Scope:** This note covers credential injection using runas.exe, DNS configuration for domain resolution, SYSVOL verification to test credentials, and understanding Kerberos vs NTLM authentication based on hostname vs IP usage.

**Prerequisites:**

- [[cards/active-directory/Active Directory\|Active Directory]]
- [[cards/red-team/Breaching Active Directory\|Breaching Active Directory]]
- [[cards/active-directory/SYSVOL\|SYSVOL]]
- [[Kerberos Authentication\|Kerberos Authentication]]
- Valid AD credentials (username and password)
- Non-domain-joined Windows machine with network access to domain
- Access to Domain Controller IP address

**Risks & Limitations:**

- Requires valid AD credentials (username and password, not just hash)
- /netonly does not authenticate against DC during injection (credentials not validated until network use)
- Local commands still run under original user context
- DNS must be properly configured to resolve domain FQDNs
- Credentials stored in process memory (vulnerable to memory dumping)
- New command prompt window required for each credential set

## How It Works (2-4 lines)
---
The runas.exe Windows built-in utility allows executing programs under different user credentials, typically requiring local or domain authentication. Security assessments often provide network access and discovered AD credentials but no means to create domain-joined machines or authenticate locally. The /netonly flag instructs runas to load credentials for network authentication only without authenticating against a domain controller, storing credentials in the spawned process's memory and using them for any outbound network connections (SMB, RPC, LDAP) while local commands execute under the original user context. Attackers leverage this to authenticate to AD resources (SYSVOL, Domain Controllers, file shares) from non-domain-joined attack machines, using either FQDNs for Kerberos authentication or IP addresses to force NTLM authentication for stealth during Red Team assessments.

## Steps (Hands-On)
---

**Legend:**

- `{DOMAIN}` = za.tryhackme.com
- `{DOMAIN_NETBIOS}` = za
- `{AD_USERNAME}` = discovered username
- `{AD_PASSWORD}` = discovered password
- `{DC_IP}` = Domain Controller IP address
- `{NETWORK_INTERFACE}` = Ethernet (or your interface name)

**Context:** You have obtained valid AD credentials during a security assessment but are on a non-domain-joined Windows machine. You need to authenticate to AD resources for enumeration and exploitation.

---

### 1. Inject AD Credentials into Memory

**Action:**

- Use runas.exe with /netonly flag to inject credentials:
```C
runas.exe /netonly /user:{DOMAIN}\{AD_USERNAME} cmd.exe
```

- Provide `{AD_PASSWORD}` when prompted

**Observable:**

- `cmd.exe` process execution
- `runas.exe` child process spawned
- Event ID 4688 (Process creation):
  - Process: `runas.exe`
  - CommandLine: `/netonly /user:{DOMAIN}\{AD_USERNAME} cmd.exe`
- New `cmd.exe` process spawned with injected credentials in memory
- Event ID 4648 (Logon using explicit credentials):
  - Subject: Original user account
  - Target: `{AD_USERNAME}@{DOMAIN}`
  - Logon Type: NewCredentials (9)
- No actual authentication to DC occurs yet (credentials only stored in memory)
- Local `whoami` command still shows original user (not `{AD_USERNAME}`)
- Process tree: original `cmd.exe` -> `runas.exe` -> new `cmd.exe` (with injected creds)

**Note:** If running from elevated cmd, local commands in new window run elevated; network authentication still uses injected AD credentials.

---

### 2. Configure DNS (Non-Domain-Joined Only)

**Action:**

- Configure DNS to point to Domain Controller for FQDN resolution:
```PowerShell
$dnsip = "{DC_IP}"
```
```PowerShell
$index = Get-NetAdapter -Name '{NETWORK_INTERFACE}' | Select-Object -ExpandProperty 'ifIndex'
```
```PowerShell
Set-DnsClientServerAddress -InterfaceIndex $index -ServerAddresses $dnsip
```

**Observable:**

- `powershell.exe` process execution
- PowerShell cmdlet execution (Get-NetAdapter, Set-DnsClientServerAddress)
- Event ID 4688 (Process creation) - powershell.exe
- Registry modification under `HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces\{InterfaceGUID}`
- DNS server setting changed to `{DC_IP}`
- Network adapter configuration modified
- Sysmon Event ID 12 (RegistryEvent) - DNS configuration change

**Note:** On domain-joined machines or when DNS configured via DHCP/VPN, this step may be automatic.

---

### 3. Verify DNS Resolution

**Action:**

- Test DNS resolution to domain FQDN:
```PowerShell
nslookup {DOMAIN}
```

**Observable:**

- `nslookup.exe` process execution
- DNS query to `{DC_IP}` on port 53 (UDP)
- Event ID 4688 (Process creation) - nslookup.exe
- DNS response from `{DC_IP}` resolving `{DOMAIN}` to DC IP address
- Sysmon Event ID 22 (DNSEvent) - DNS query for `{DOMAIN}`
- Network connection to DNS server (port 53)
- Output should show:
```
  Server:  {DOMAIN}
  Address: {DC_IP}
  
  Name:    {DOMAIN}
  Address: {DC_IP}
```

---

### 4. Verify Credentials via SYSVOL Access

**Action:**

- Force network-based listing of SYSVOL to test injected credentials:
```PowerShell
dir \\{DOMAIN}\SYSVOL\
```

**Observable:**

- SMB connection to Domain Controller on port 445
- DNS resolution of `{DOMAIN}` to `{DC_IP}`
- Kerberos authentication attempt (AS-REQ/AS-REP) since FQDN used
- Event ID 4768 (Kerberos TGT requested) on DC:
  - Account: `{AD_USERNAME}`
  - Source IP: Attacker machine IP
  - Pre-authentication Type: Encrypted Timestamp
- Event ID 4624 (Logon) on DC:
  - Logon Type 3 (Network)
  - Account: `{AD_USERNAME}@{DOMAIN}`
  - Source: Attacker machine
  - Authentication Package: Kerberos
- Successful directory listing of SYSVOL contents
- Sysmon Event ID 3 (Network connection) - SMB to DC port 445
- SYSVOL directory structure displayed (proves credentials valid)

**Note:** Any AD account, no matter how low-privileged, can read SYSVOL contents. This is ideal for credential verification.

---

### 5. Understanding Hostname vs IP Authentication

**Action:**

- Compare authentication types:
  - Hostname: `dir \\{DOMAIN}\SYSVOL\` (Kerberos)
  - IP: `dir \\{DC_IP}\SYSVOL\` (NTLM)

**Observable:**

**Using Hostname (`\\{DOMAIN}\SYSVOL\`):**
- DNS query to resolve `{DOMAIN}`
- Kerberos authentication (AS-REQ/AS-REP on port 88)
- Event ID 4768 (Kerberos TGT requested) on DC
- Hostname embedded in Kerberos tickets
- More likely monitored by Blue Team for Pass-the-Hash/OverPass-the-Hash

**Using IP (`\\{DC_IP}\SYSVOL\`):**
- No DNS query (direct IP connection)
- NTLM authentication forced (no Kerberos)
- Event ID 4776 (NTLM authentication) on DC
- No hostname in authentication protocol
- Potentially stealthier during Red Team assessment

**Why This Matters:**

Organizations monitoring for OverPass-the-Hash and Pass-the-Hash attacks may focus on Kerberos authentication patterns. Forcing NTLM by using IP addresses can be a stealth tactic.

---

### 6. Use Injected Credentials for Applications

**Action:**

- Launch network-authenticating applications from runas command prompt:
  - MS SQL Server Management Studio
  - Web browsers (for NTLM-authenticated web apps)
  - Any application requiring domain authentication

**Observable:**

- Application process spawned from runas cmd.exe window
- Application GUI may show local username
- Actual network authentication uses `{AD_USERNAME}` credentials
- Event ID 4624 (Logon) on target server:
  - Logon Type 3 (Network)
  - Account: `{AD_USERNAME}@{DOMAIN}`
- Authentication successful despite local GUI showing different user
- Network connections authenticated with injected credentials
- No credential prompts required for domain resources

**Example Use Cases:**

- MS SQL Studio with Windows Authentication (shows local user but authenticates as AD user)
- Web applications using NTLM authentication
- File shares requiring domain credentials
- RDP to domain systems
- Any network service supporting integrated Windows authentication
## Blue - Detection & Response
---

**Indicators of Compromise:**

- Event ID 4648 (Logon using explicit credentials) - runas.exe with /netonly flag
- Event ID 4688 (Process creation) - runas.exe with `/netonly /user:DOMAIN\` in CommandLine
- Process tree showing cmd.exe spawning runas.exe spawning another cmd.exe
- Event ID 4624 (Logon Type 9 - NewCredentials) on workstation
- Event ID 4768 (Kerberos TGT request) from non-domain-joined or unusual IP addresses
- Event ID 4624 (Logon Type 3) on DC from workstations not typically authenticating
- DNS queries to Domain Controller from non-standard systems
- SYSVOL access from non-domain-joined machines or unusual IPs
- Multiple applications authenticating as same AD user from single non-domain workstation
- NTLM authentication (Event ID 4776) from systems typically using Kerberos
- SMB connections to DC from non-domain-joined systems
- Runas.exe execution with /netonly flag during non-business hours
- Network authentication by user accounts not assigned to source system
- Credential injection followed by immediate AD enumeration activity

![cards/red-team/images/Enumerating Active Directory.png|500](/img/user/cards/red-team/images/Enumerating%20Active%20Directory.png)