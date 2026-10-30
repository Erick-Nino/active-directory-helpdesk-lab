# Enterprise Active Directory & IT Support Operations Lab

## Executive Summary
Designed and deployed an enterprise virtual domain environment replicating corporate IT support and administration. The infrastructure features a Windows Server 2025 Domain Controller hosting the lab.local root domain, with two domain-joined Windows 11 endpoints: a dedicated Help Desk workstation equipped with Remote Server Administration Tools (RSAT) and a standard employee endpoint (Patty). 

The lab demonstrates foundational system administration and Tier 1/2 support: implementing departmental Organizational Units (IT and HR), configuring Role-Based Access Control (RBAC), provisioning secured SMB network shares (HR and Personal), administering Active Directory remotely via RSAT, and resolving simulated help desk tickets using Spiceworks.

---

## Core Competencies & Environment

| Category | Technologies / Implementations |
| :--- | :--- |
| **Operating Systems** | Windows Server 2025, Windows 11 Pro/Enterprise |
| **Directory Services** | Active Directory Domain Services (AD DS), DNS, DHCP |
| **Remote Administration** | Remote Server Administration Tools (RSAT) — ADUC, GPMC, DNS Manager |
| **Network Storage** | SMB Network Shares (HR Department Share, Personal User Storage), NTFS & Share Permissions |
| **Service Management** | Spiceworks Help Desk (Ticket triaging, SLA tracking, resolution documentation) |
| **Identity & Security** | Role-Based Access Control (RBAC), Least-Privilege Administration, Domain Trust Management |

---

## Network & Architecture Map

                                  [ lab.local Domain ]
                                           |
                 +-------------------------+-------------------------+
                 |                                                   |
       [ Domain Controller ]                                 [ Internal Network ]
       Hostname: DC01                                        Subnet: 10.1.10.0/24
       OS: Windows Server 2025                               Gateway: 10.1.10.1
       IP: 10.1.10.2 (Static)                                DNS: 10.1.10.2
       Roles: AD DS, DNS, File Server                        Shares: \HR, \Personal
                 |                                                   |
                 +-------------------------+-------------------------+
                                           |
                      +--------------------+--------------------+
                      |                                         |
           [ Help Desk Workstation ]                 [ Standard User Client ]
           Hostname: HELPDESK-PC                     Hostname: PATTY-PC
           OS: Windows 11                            OS: Windows 11
           IP: 10.1.10.3 (Static/Assigned)           IP: 10.1.10.4 (DHCP)
           Role: IT Support / RSAT Console           User: Patty (HR Department)
Implementation Details
1. Domain Architecture & Departmental Structure
Installed and configured AD DS on Windows Server 2025, provisioning the lab.local forest root.

Designed an enterprise Organizational Unit (OU) schema:

IT OU: Hosts administrative accounts, tier-specific support roles, and IT workstations.

HR OU: Contains standard staff accounts (including test user Patty) and departmental assets.

Enforced centralized domain security policies: password complexity, account lockout thresholds after repeated invalid attempts, and mandatory first-login password updates.

2. Remote Server Administration (RSAT) & Least Privilege
Configured a dedicated Windows 11 Help Desk PC with Remote Server Administration Tools (RSAT).

Avoided direct Domain Controller interactive logins (enterprise best practice) by managing user lifecycles, group memberships, and computer accounts remotely via RSAT from the Help Desk machine.

Configured delegated administration in Active Directory Users and Computers (ADUC), giving the Help Desk account the exact rights needed to reset passwords and unlock accounts without full Domain Admin rights.

Restricted the standard user account (Patty) to non-privileged local access to prevent unauthorized system modifications.

3. Network File Shares & Security Permissions
Configured two centralized SMB shares on the server with tailored Share and NTFS permissions:

HR Share: Restricted strictly to members of the HR-Department security group via NTFS access lists to prevent unauthorized cross-departmental data access.

Personal Share: Provisioned as dedicated individual workspace storage for user document backups.

Configured automated drive mapping via Group Policy Objects (GPOs) to mount these shares automatically upon user login.

Incident Management & Simulated Tickets (Spiceworks)
All simulated technical issues were logged, triaged, and documented within Spiceworks Help Desk:

Ticket	Requester	Issue Description	Root Cause	Resolution & Verification
#101	Patty (HR)	Account locked out after returning from leave; cannot log in to Windows 11.	Failed login attempts exceeded the domain account lockout threshold.	Verified employee identity. Accessed ADUC via RSAT on Helpdesk PC, unlocked account, and set a temporary password with "User must change password at next logon." Verified successful login on PATTY-PC.
#102	Patty (HR)	"Access Denied" error when attempting to open the HR network folder.	User was omitted from the HR-Department global security group.	Added Patty's account to the HR-Department group using RSAT. Executed gpupdate /force and had user re-authenticate to refresh Kerberos security tokens. Verified full read/write access to the HR share.
#103	Patty (HR)	Workstation boots with: "The trust relationship between this workstation and the primary domain failed."	Secure channel password out of sync between PATTY-PC and the domain controller.	Reverted client to local workgroup, reset the computer object in ADUC, and rejoined PATTY-PC to lab.local. Confirmed successful domain authentication with Test-ComputerSecureChannel.
#104	Patty (HR)	Client unable to connect to shared drives or resolve internal intranet names.	Client network adapter had manual public DNS configured, preventing internal lab.local SRV record lookups.	Reconfigured adapter to pull DNS automatically (10.1.10.2). Flushed local resolver cache with ipconfig /flushdns and registered DNS with ipconfig /registerdns. Internal connectivity restored.
Key Technical Takeaways
Remote Administration Hygiene: Demonstrated industry-standard management by administering Active Directory remotely via RSAT on a Windows 11 client rather than opening direct administrative sessions on the production Domain Controller.

Data Protection via NTFS: Applied defense-in-depth by combining SMB share permissions with granular NTFS security descriptors to safeguard departmental (HR) data.

Structured Troubleshooting: Utilized systematic diagnostics (ipconfig, ping, gpupdate, Test-ComputerSecureChannel) paired with ticketing documentation to verify resolutions before closing requests.
