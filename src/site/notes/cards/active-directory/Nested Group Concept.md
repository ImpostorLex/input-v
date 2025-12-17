---
{"dg-publish":true,"permalink":"/cards/active-directory/nested-group-concept/","tags":["windows/ad/concept"]}
---

~ [[cards/active-directory/windows active directory\|Active Directory]]
## How it Works
---

**Group Nesting in Organizations:**

In many organizations, **group nesting** is used to simplify permissions. For example:
```
IT Support (Parent Group)
    ├── Helpdesk
    ├── Access Card Managers
    └── Network Managers
```

Each of these subgroups inherits permissions of **IT Support** but can also have more specific permissions.

**Privileged Group Monitoring:**

Privileged groups are monitored more closely for changes than others. Any group that classifies as protected group, such as Domain Admins or Enterprise Admins.

**Examples of Less-Monitored Groups (Microsoft standard AD setup):**

- **The IT Support group** - Can be used to gain privileges such as force changing user passwords. Although in most cases you won't be able to reset passwords of privileged users, having ability to reset even low-privileged users allows spreading to workstations.

- **Local Admin Rights and Monitoring Gaps** - Groups with local admin rights are often less monitored, making them prime targets for attackers. Gaining access through network support group can provide persistent access to critical systems, enabling future domain compromises.

- **Indirect Privileges via GPO Ownership** - Persistence isn't always about direct admin rights. Groups with control over Group Policy Objects (GPOs) can also be used for long-term access, as GPOs manage critical network settings that can be manipulated for malicious purposes.

**Visibility Issues:**

The challenge with nested groups is that it reduces **visibility** into actual access. For example, if you check membership for **IT Support**, it might show just 3 groups (Helpdesk, Access Card Managers, and Network Managers), but each of those could contain additional subgroups. So to find real access, you'd need to check each level.
```
IT Support
    ├── Helpdesk
    ├── Access Card Managers
    └── Network Managers
            └── Admin Subgroup (privileged)
```

To fully understand access, you'd need to enumerate every subgroup, and potentially sub-subgroups. This becomes increasingly complex with deeper nesting.

**Monitoring Gaps:**

This nesting can also lead to **monitoring gaps**. Let's say there's an alert for when someone is added to **Domain Admins**. This **alert won't fire** if a user is added to a **subgroup** of **Domain Admins**, like a nested group called **Domain Admins - Network Managers**.
```
Domain Admins
    └── Network Managers (privileged)
```

Because **AD management** (Active Directory) and **InfoSec teams** often work separately, they may miss changes in subgroups, leaving security gaps.