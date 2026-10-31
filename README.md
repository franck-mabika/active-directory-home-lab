# Active Directory Home Lab

A small company network built on my laptop with VirtualBox: a **Windows Server 2022 domain controller** managing users, groups, file-share permissions and security policies for a **Windows 11** workstation.

**Tools:** Oracle VirtualBox 7.1 · Windows Server 2022 (evaluation) · Windows 11 Enterprise (evaluation)

## Skills demonstrated
- Virtualization and virtual networking (VirtualBox NAT Network)
- Windows Server installation and static IP configuration
- Active Directory Domain Services: new forest, domain controller promotion
- DNS for a domain, joining a workstation to the domain
- Organizational Units, security groups and user accounts
- Share and NTFS permissions following **least privilege**
- Group Policy: password and account lockout policy
- Troubleshooting: networking, DNS, permissions, host resources
- Helpdesk task: unlocking a locked-out account

## Lab architecture
![Network diagram](01-network-diagram.png)

| VM | OS | Role | IP |
|---|---|---|---|
| DC01 | Windows Server 2022 Standard (Desktop Experience) | Domain controller + DNS for `lab.local` | 10.0.2.10 (static) |
| PC01 | Windows 11 Enterprise | Domain-joined workstation | 10.0.2.4 (DHCP), DNS 10.0.2.10 |

| OU | Security group | User | Shared folder |
|---|---|---|---|
| Sales | Sales Team | jdoe | `\\DC01\Sales` |
| HR | HR Team | mlee | `\\DC01\HR` |
| IT | IT Admins | aruiz | `\\DC01\IT` |

## Implementation

### 1. Network and server
Created the LabNet NAT Network (10.0.2.0/24), installed Windows Server 2022 on DC01 and gave it a static IP, with its own address as DNS.

![DC01 static IP](02-dc01-static-ip.png)

### 2. Active Directory domain
Installed the AD DS role and promoted DC01 to the domain controller of a new forest, `lab.local`.

![Domain controller promoted](03-dc-promoted.png)
![LAB\Administrator login](04-lab-administrator-login.png)

### 3. Organizational structure
One OU per department, a security group in each, and one user per department added to their group.

![OU, group and user](05-ou-group-user.png)

### 4. Joining PC01 to the domain
Pointed PC01's DNS at DC01, confirmed name resolution, then joined the domain.

![nslookup lab.local](06-nslookup.png)
![Welcome to the lab.local domain](07-domain-join.png)
![PC01 in Computers](08-pc01-in-computers.png)

### 5. Department shared folders (least privilege)
For each of `C:\Shares\Sales`, `HR` and `IT`: shared with only the department's group (Change), NTFS inheritance disabled, Users removed, the group given Modify.

![IT NTFS permissions](09-it-ntfs-permissions.png)

### 6. Password and lockout policy
Default Domain Policy: minimum password length **12** (complexity enabled), lockout after **5** invalid attempts for 30 minutes. Applied with `gpupdate /force`.

![Password policy](10-password-policy.png)
![Lockout policy](11-lockout-policy.png)

## Results
All 9 user-to-folder combinations tested from PC01:

| User | Sales | HR | IT |
|---|---|---|---|
| jdoe (Sales) | ✅ Access | ❌ Denied | ❌ Denied |
| mlee (HR) | ❌ Denied | ✅ Access | ❌ Denied |
| aruiz (IT) | ❌ Denied | ❌ Denied | ✅ Access |

![Jane opens Sales](12-jane-sales-access.png)
![Mark denied on Sales](13-mark-sales-denied.png)
![Ana opens IT](14-ana-it-access.png)

**Lockout test:** after 5 wrong passwords, `LAB\mlee` was locked out. I unlocked it in Active Directory Users and Computers and the user could log in again.

![Mark locked out](15-mark-locked-out.png)

## Challenges and solutions
| Problem | Cause | Fix |
|---|---|---|
| VMs aborted mid-install | Laptop disk nearly full | Freed space from 10 GB to 46 GB; deleted installer ISOs |
| PC01 network "Name" box greyed out | Adapter on NAT, not NAT Network | Switched to NAT Network → LabNet |
| PC01 couldn't resolve `lab.local` | DC01 freezing, host RAM at 87% | Restarted the host, gave DC01 less RAM |
| DC01 aborted when resuming | Corrupted saved state | Discarded the saved state; domain intact |
| OU and group names swapped | Renamed the wrong object | Renamed each back |
| `\\DC01\IT` "name incorrect" | Permissions applied to the parent `Shares` folder | Reset Shares, recreated IT, checked the window title |

![Problem: NAT instead of NAT Network](16-problem-nat.png)
![Problem: VMs aborted](17-problem-aborted.png)
![Problem: permissions on the parent folder](18-problem-it-on-shares.png)

## Next steps
- Automate user creation with PowerShell
- Send DC01's security logs to a SIEM (Wazuh) and detect the lockout
