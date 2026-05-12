# Lab 1 — Active Directory
### Windows Server 2025 · Azure Free Account · Identity & Access Management
---

https://www.loom.com/share/598706fa4c9c433181072ba79a01366f

---

| Field | Value |
|---|---|
| **Certification alignment** | CompTIA Network+ · Security+ · Azure Administrator |
| **Free tools** | Windows Server 2025 Evaluation (180 days) · Azure Free Account |
| **Time to complete** | 3–5 hours across multiple sessions |
| **Estimated cost** | $0 — fully covered by free tiers and evaluation licences |
| **Career relevance** | IT Support · Sysadmin · Cloud Engineer · Security Analyst |

---

## Overview

Every organisation running Windows infrastructure relies on Active Directory to answer one fundamental question: **who is allowed to do what?**

Active Directory is the identity backbone. It controls which users log into which computers, which groups access which file shares, and which policies apply across the organisation. When a new employee joins, IT creates their account, adds them to groups, and access is granted automatically. When they leave, disabling one account closes every door simultaneously.

This is not legacy technology. Hybrid environments sync on-premises Active Directory to **Microsoft Entra ID** (formerly Azure AD) in the cloud. On-prem AD knowledge transfers directly to cloud roles.

| Role | How this lab applies |
|---|---|
| IT Support / Help Desk | Password resets, account unlocks, group membership changes — the top three ticket types in any enterprise |
| Sysadmin | Designing OU structure, deploying GPOs, managing domain-joined machines at scale |
| Cloud Engineer | Entra ID uses the same concepts: users, groups, roles, conditional access. On-prem AD knowledge transfers directly |
| Security Analyst | AD is the most targeted system in ransomware attacks. Understanding how it works is the foundation of defending it |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      FOREST: lab.local                      │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                 DOMAIN: lab.local                   │   │
│   │                                                     │   │
│   │   ┌──────────────────────────────────────────────┐  │   │
│   │   │         Domain Controller (DC01)             │  │   │
│   │   │  • Active Directory Domain Services          │  │   │
│   │   │  • DNS Server                                │  │   │
│   │   │  • Group Policy Management                   │  │   │
│   │   └──────────────────────────────────────────────┘  │   │
│   │                        │                            │   │
│   │         ┌──────────────┼──────────────┐            │   │
│   │         ▼              ▼              ▼            │   │
│   │   ┌──────────┐  ┌──────────┐  ┌──────────┐        │   │
│   │   │  OU: IT  │  │OU:Finance│  │  OU: HR  │  ...   │   │
│   │   │          │  │          │  │          │        │   │
│   │   │IT_Admins │  │ Finance  │  │ HR_Users │        │   │
│   │   │ (group)  │  │  Users   │  │ (group)  │        │   │
│   │   │          │  │ (group)  │  │          │        │   │
│   │   │alice.chen│  │bob.patel │  │carol.jones│       │   │
│   │   └──────────┘  └──────────┘  └──────────┘        │   │
│   │                                                     │   │
│   │   ┌──────────────────────────────────────────────┐  │   │
│   │   │       GPO: IT Security Policy                │  │   │
│   │   │  • Min password: 12 chars + complexity       │  │   │
│   │   │  • Screen lock: 15 minutes                   │  │   │
│   │   │  • USB access: Denied                        │  │   │
│   │   └──────────────────────────────────────────────┘  │   │
│   └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘

              ┌─────────────────────────────┐
              │   Domain-joined Workstation │
              │   (joins via lab.local)     │
              │   GPO applied automatically │
              └─────────────────────────────┘
```

> **Authentication flow**: When a user logs in to any domain-joined machine → credentials are sent to the Domain Controller → DC validates the password hash → DC returns an access token containing the user's group memberships → Windows uses the token to determine what resources the user can access.

---

## What You Will Learn

| Skill | Real-world application |
|---|---|
| Promote a Windows Server to Domain Controller | The first step in every enterprise Windows environment |
| Create Organisational Units (OUs) | Apply different policies to different departments |
| Create users, groups, and group memberships | Every access decision in an enterprise is group-based |
| Configure Group Policy Objects (GPOs) | Enforce settings centrally across every machine in the domain |
| Join a machine to the domain | Connect a workstation so it becomes a managed, policy-enforced resource |
| Configure role-based access with security groups | Apply the principle of least privilege at scale |
| Reset passwords and manage account lifecycle | The most frequent real-world task for IT support |

---

## Prerequisites

- An Azure free account — [azure.microsoft.com/free](https://azure.microsoft.com/free) — **OR** a local machine with 8 GB RAM for VirtualBox
- No prior Active Directory experience required
- A local RDP client (Windows built-in, or Microsoft Remote Desktop on macOS)

---

## Lab Environment Setup

### Option A — Run in Azure (Recommended)

No local hardware requirements. The VM runs in Microsoft's data centre; you connect via RDP.

1. Go to [azure.microsoft.com/free](https://azure.microsoft.com/free) and create a free account
2. Sign in to [portal.azure.com](https://portal.azure.com)
3. Search for **Virtual machines** and click **Create**
4. Configure the VM using the settings below
5. Click **Review + Create**, then **Create**

| Setting | Value | Why |
|---|---|---|
| Region | East US | Cheapest region, most available VM sizes under free tier |
| Image | Windows Server 2025 Datacenter — Gen2 | Latest server OS, includes free 180-day evaluation licence |
| Size | Standard_B2s (2 vCPU, 4 GB RAM) | Smallest size that runs AD comfortably. Covered by free tier credits |
| Authentication | Password | Set a strong password — you will use this to RDP in |
| Public inbound ports | Allow RDP (3389) | Required to connect from your local machine |
| OS disk | Standard SSD | Good performance, included in free tier storage |

> ⚠️ **Stop the VM when not in use.** A B2s VM costs ~$0.05/hour. Stopping (not deleting) pauses compute billing. Your $200 free credit lasts much longer if you stop the VM at the end of every session.

#### Fix: Enable clipboard sharing over RDP

By default, RDP does not share your clipboard. Fix this before connecting:

1. Open the Remote Desktop application on your local machine
2. Enter the VM's public IP address
3. Click **Show Options** → **Local Resources** tab
4. Check **Clipboard** under *Local devices and resources*
5. Click **Connect**

> If connecting through the Azure portal browser console, clipboard sharing is limited. Download the RDP file from the Azure portal (**Connect → Download RDP File**) and open it with the native Remote Desktop app instead. This is the recommended approach for all lab work.

---

### Option B — Run Locally with VirtualBox

1. Download VirtualBox from [virtualbox.org](https://www.virtualbox.org) — free, no account required
2. Download the Windows Server 2025 Evaluation ISO from [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025)
3. Create a new VM: 4 GB RAM minimum, 60 GB disk, **Windows Server 2019/2022** as the type
4. Mount the ISO and boot the VM — follow the installation wizard
5. Select **Windows Server 2025 Datacenter with Desktop Experience** during setup

> **Minimum local hardware**: 8 GB RAM (4 GB for VM, 4 GB for your OS) · 60 GB free disk space · Quad-core CPU with virtualisation enabled in BIOS. If your machine has less than 8 GB RAM, use the Azure option.

---

## Step 1 — Install Active Directory Domain Services

RDP into your Windows Server VM. Open **Server Manager** — it opens automatically on login. All configuration is done inside the VM.

> **What is a Domain Controller?**
> A Domain Controller (DC) is a server that runs Active Directory. It is the brain of the entire identity system. When a user logs in anywhere on the domain, their credentials are checked against the DC. Everything that joins your network trusts this server to make authentication decisions.

**GUI method:**

1. In Server Manager, click **Manage → Add Roles and Features**
2. Click **Next** through the wizard until you reach **Server Roles**
3. Check **Active Directory Domain Services**
4. When prompted, click **Add Features** to include the management tools
5. Click **Next** through remaining pages and click **Install**
6. Wait 2–3 minutes for installation to complete, then click **Close** — do not restart yet

**PowerShell equivalent:**
```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

### Also install Group Policy Management Console (GPMC) now

Step 5 requires the Group Policy Management Console — a separate tool from Active Directory Users and Computers. Install it now so it is ready when you need it.

```powershell
# Run immediately after installing AD DS
Install-WindowsFeature -Name GPMC
# Wait for completion, then close and reopen Server Manager
```

Once installed, **Group Policy Management** appears in the **Tools** menu in Server Manager. This is a completely separate window from Active Directory Users and Computers — do not look for GPOs inside ADUC.

---

## Step 2 — Promote the Server to a Domain Controller

Installing the AD DS role does not create a domain. Promotion creates your forest, domain, and makes this server the authoritative DNS and identity server for everything that joins it.

> **What is a Forest and Domain?**
> A **Forest** is the top-level container of your entire Active Directory structure — think of it as the organisation itself. A **Domain** is a boundary inside the forest with a name (ours is `lab.local`). Most small-to-medium organisations have one domain inside one forest.

**GUI method:**

1. In Server Manager, click the **yellow warning flag** at the top right
2. Click **Promote this server to a domain controller**
3. Select **Add a new forest**
4. Set **Root domain name** to: `lab.local`
5. Click **Next** — set a DSRM password (write it down — needed for disaster recovery)
6. Click through DNS Options and NetBIOS pages — accept the defaults
7. Click **Install** — the server will automatically restart when complete

**PowerShell equivalent:**
```powershell
Import-Module ADDSDeployment
Install-ADDSForest `
  -DomainName 'lab.local' `
  -DomainNetBiosName 'LAB' `
  -InstallDns:$true `
  -SafeModeAdministratorPassword (ConvertTo-SecureString 'YourDSRMPassword!' -AsPlainText -Force) `
  -Force:$true
```

> After promotion, this server runs DNS for `lab.local` and is the authoritative source for all identity decisions. Every machine that joins `lab.local` trusts this server to authenticate users.

---

## Step 3 — Build the Organisational Structure

Open **Active Directory Users and Computers (ADUC)** from the Tools menu in Server Manager.

### Organisational Units (OUs)

> **What is an OU?**
> An Organisational Unit is a folder inside Active Directory. You use OUs to organise users, computers, and groups by department or function. The real power: you can link a Group Policy to an OU, so every user or computer inside automatically gets those policies applied.

**GUI:** Right-click `lab.local` in ADUC → **New → Organizational Unit**

**PowerShell — create all 5 OUs at once:**
```powershell
New-ADOrganizationalUnit -Name "IT"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Finance"   -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "HR"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Sales"     -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Computers" -Path "DC=lab,DC=local"
```

### Security Groups

> **What is a Security Group?**
> Instead of granting access to individual users, you grant access to a group and add users to it. This is role-based access control. Adding 50 Finance employees to a new system means adding the `Finance_Users` group once — not 50 individual accounts.

**GUI:** Right-click each OU → **New → Group**. Set Group scope to **Global** and Group type to **Security**.

```powershell
New-ADGroup -Name "IT_Admins"     -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=lab,DC=local"
New-ADGroup -Name "Finance_Users" -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=lab,DC=local"
New-ADGroup -Name "HR_Users"      -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=lab,DC=local"
New-ADGroup -Name "Sales_Users"   -GroupScope Global -GroupCategory Security -Path "OU=Sales,DC=lab,DC=local"
```

### User Accounts

> **What is a User Account?**
> A User Account represents a person in Active Directory. It holds the username, password hash, group memberships, and attributes. When someone logs into a domain-joined machine, Windows sends credentials to the DC, which returns a token listing what the user can access — all based on group membership.

> ⚠️ **Run the block below all at once — not line by line.** The `$password` variable must be defined before the `New-ADUser` commands. Select all, then press **F8** in PowerShell ISE, or paste the entire block into a PowerShell window and press **Enter**.

```powershell
# Step 1 — define the password variable first
$password = ConvertTo-SecureString "Welcome@2026!" -AsPlainText -Force

# Step 2 — create all 4 users
New-ADUser -Name "alice.chen" -GivenName "Alice" -Surname "Chen" `
  -SamAccountName "alice.chen" -UserPrincipalName "alice.chen@lab.local" `
  -Path "OU=IT,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "bob.patel" -GivenName "Bob" -Surname "Patel" `
  -SamAccountName "bob.patel" -UserPrincipalName "bob.patel@lab.local" `
  -Path "OU=Finance,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "carol.jones" -GivenName "Carol" -Surname "Jones" `
  -SamAccountName "carol.jones" -UserPrincipalName "carol.jones@lab.local" `
  -Path "OU=HR,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "david.smith" -GivenName "David" -Surname "Smith" `
  -SamAccountName "david.smith" -UserPrincipalName "david.smith@lab.local" `
  -Path "OU=Sales,DC=lab,DC=local" -AccountPassword $password -Enabled $true

# Step 3 — add each user to their department group
Add-ADGroupMember -Identity "IT_Admins"     -Members "alice.chen"
Add-ADGroupMember -Identity "Finance_Users" -Members "bob.patel"
Add-ADGroupMember -Identity "HR_Users"      -Members "carol.jones"
Add-ADGroupMember -Identity "Sales_Users"   -Members "david.smith"
```

---

## Step 4 — Configure Group Policy

Open **Group Policy Management** from the Tools menu in Server Manager.

> **What is a GPO?**
> A Group Policy Object is a collection of settings applied automatically to every user or computer inside an OU. Create one GPO, link it to an OU, and every machine and user in that OU gets those rules applied the next time they log in or run `gpupdate`. Password policies, screen lock timers, USB restrictions — all enforced from one central place.

**Create and configure the IT Security Policy:**

1. Expand **Forest: lab.local → Domains → lab.local** in Group Policy Management
2. Right-click the **IT** OU → **Create a GPO in this domain and link it here**
3. Name it: `IT Security Policy`
4. Right-click the new GPO → **Edit**
5. Configure the following settings:

| Policy path | Setting | Value | Why |
|---|---|---|---|
| Computer Config → Windows Settings → Security → Account Policies → Password Policy | Minimum password length | 12 | Enforces strong passwords across all IT accounts |
| Computer Config → Windows Settings → Security → Account Policies → Password Policy | Password must meet complexity requirements | Enabled | Requires upper, lower, number, and symbol |
| Computer Config → Windows Settings → Security → Local Policies → Security Options | Interactive logon: Machine inactivity limit | 900 seconds | Auto-locks screen after 15 minutes |
| Computer Config → Administrative Templates → System → Removable Storage Access | All removable storage classes: Deny all access | Enabled | Prevents data exfiltration via USB drives |

> **Test your GPO:** Join a second VM to `lab.local`, move its computer account into the IT OU, run `gpupdate /force`, log in as `alice.chen`, and verify the screen lock policy takes effect.

---

## Step 5 — Common Help Desk Tasks

Practice each task on your test accounts. These are the most frequent real-world operations for any IT support role.

### Reset a password

```powershell
Set-ADAccountPassword -Identity "bob.patel" -Reset `
  -NewPassword (ConvertTo-SecureString "NewPass@2026!" -AsPlainText -Force)
Set-ADUser -Identity "bob.patel" -ChangePasswordAtLogon $true
```

> Always force a password change on next login so the user sets their own password immediately.

### Unlock a locked account

```powershell
Unlock-ADAccount -Identity "carol.jones"
```

### Disable an account (offboarding)

```powershell
# Disable an account — preserves history and group memberships for audit
Disable-ADAccount -Identity "david.smith"

# Find all currently disabled accounts
Search-ADAccount -AccountDisabled | Select-Object Name, SamAccountName
```

> Disable rather than delete. Disabling preserves account history and group memberships in case they are needed for audit purposes.

### Audit and reporting

```powershell
# Find accounts inactive for 90+ days
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true} `
  -Properties LastLogonDate | Select-Object Name, LastLogonDate

# Check group membership for a specific user
Get-ADPrincipalGroupMembership -Identity "alice.chen" | Select-Object Name
```

---

## Verification

Confirm the lab is working correctly with these checks:

| Check | Command | Expected result |
|---|---|---|
| Domain controller is running | `Get-ADDomainController` | Returns DC info including forest `lab.local` |
| OUs exist | `Get-ADOrganizationalUnit -Filter *` | Lists all 5 OUs you created |
| Users exist and are enabled | `Get-ADUser -Filter {Enabled -eq $true}` | Lists your 4 test accounts |
| Group memberships correct | `Get-ADGroupMember -Identity IT_Admins` | Returns `alice.chen` |
| GPO is linked | `Get-GPInheritance -Target 'OU=IT,DC=lab,DC=local'` | Shows `IT Security Policy` as linked |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| PowerShell prompts for `Name:` when creating users | You ran `New-ADUser` before defining `$password`. Run the entire script block at once — the `$password` line must come first. |
| Cannot copy and paste into the VM | Open the RDP client → **Show Options → Local Resources** → check **Clipboard**. Reconnect. Or download the RDP file from the Azure portal and use the native Remote Desktop app. |
| Promotion fails: DNS conflict | Set the NIC's preferred DNS to `127.0.0.1` before promoting, or use the static IP of the VM |
| Cannot RDP after domain join | Log in as `LAB\Administrator` (domain admin), not just `Administrator` |
| GPO not applying | Run `gpupdate /force` on the target machine, then `gpresult /r` to see applied policies |
| User cannot log in after creation | Confirm the account is **Enabled** and `ChangePasswordAtLogon` is set correctly |
| AD Users and Computers not showing | Run `dsa.msc` from the Run dialog, or run `Add-WindowsFeature RSAT-ADDS` |

---
