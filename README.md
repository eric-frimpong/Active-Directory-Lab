# 🏢 Active Directory Labs — OUs, Groups, Users & Account Management

> A two-part hands-on Active Directory lab series documenting enterprise directory structure setup, user management, password resets, and helpdesk workflows — built to mirror real-world IT operations.

---

## 📋 Table of Contents

1. [Lab Environment](#-lab-environment)
2. [Part 1 — AD Structure: OUs, Groups & Users (computerwiz.com)](#-part-1--ad-structure-ous-groups--users)
   - [Step 1 — Create Organizational Units](#step-1--create-organizational-units)
   - [Step 2 — Add Sub-OUs Within Each Region](#step-2--add-sub-ous-within-each-region)
   - [Step 3 — Create a Security Group](#step-3--create-a-security-group)
   - [Step 4 — Create a Distribution Group](#step-4--create-a-distribution-group)
   - [Directory Structure Summary](#directory-structure-summary)
   - [PowerShell Equivalent](#powershell-equivalent--part-1)
3. [Part 2 — Domain Controller: Users, Groups & Password Management (corp.local)](#-part-2--domain-controller-users-groups--password-management)
   - [Part 2 Environment](#part-2-environment)
   - [Section 1 — OU Structure](#section-1--ou-structure)
   - [Section 2 — User Account Management](#section-2--user-account-management)
   - [Section 3 — Security Groups & Membership](#section-3--security-groups--group-membership)
   - [Section 4 — Password Reset](#section-4--password-reset)
   - [Section 5 — Account Lockout & Unlock](#section-5--account-lockout--unlock)
   - [Section 6 — Disabling & Enabling Accounts](#section-6--disabling--enabling-accounts)
   - [Section 7 — Verification from Client Machine](#section-7--verification-from-client-machine)
   - [Summary of Tasks Completed](#-summary-of-tasks-completed)
4. [Key Concepts](#-key-concepts)

---

## 🖥️ Lab Environment

### Part 1 — computerwiz.com

| Requirement      | Details                                     |
| ---------------- | ------------------------------------------- |
| Operating System | Windows Server 2019                         |
| Role Installed   | Active Directory Domain Services (AD DS)    |
| Tool Used        | Active Directory Users and Computers (ADUC) |
| Permissions      | Domain Admin or equivalent                  |
| Domain           | `computerwiz.com`                           |

### Part 2 — corp.local

| VM           | Role                                       | RAM | Storage | Network          |
| ------------ | ------------------------------------------ | --- | ------- | ---------------- |
| WIN-DC01     | Domain Controller (Windows Server 2022)    | 4GB | 60GB    | Internal Network |
| WIN-CLIENT01 | Domain-joined Workstation (Windows 11 Pro) | 2GB | 40GB    | Internal Network |

| Setting           | Value           |
| ----------------- | --------------- |
| Domain Name       | `corp.local`    |
| DC Hostname       | `WIN-DC01`      |
| DC IP Address     | `192.168.10.1`  |
| Client IP Address | `192.168.10.10` |
| DNS Server        | `192.168.10.1`  |

**Tools used:** Active Directory Users & Computers (ADUC), Active Directory Administrative Center (ADAC), PowerShell (AD Module)

---

## 🏗️ Part 1 — AD Structure: OUs, Groups & Users

This lab simulates an enterprise Active Directory environment for a fictional company on the domain `computerwiz.com`. The goal is to organise users and computers into regional Organizational Units, create sub-OUs for object separation, and add Security and Distribution Groups to manage access and communications.

---

### Step 1 — Create Organizational Units

**Goal:** Create top-level OUs representing geographic regions — USA, Europe, and Asia.


<img width="955" height="715" alt="mdlkAvS - Imgur" src="https://github.com/user-attachments/assets/50116845-b71b-411f-9f20-4b201587b3f2" />

1. Open **Server Manager → Tools → Active Directory Users and Computers**
2. In the left pane, right-click your domain (`computerwiz.com`)
3. Navigate to **New → Organizational Unit**
4. Type the name (e.g., `USA`) and click **OK**
5. Repeat for `Europe` and `Asia`

> OUs allow you to delegate administration and apply Group Policy Objects (GPOs) to specific segments of your organisation without affecting the entire domain.

---

### Step 2 — Add Sub-OUs Within Each Region

**Goal:** Create `Computers`, `Users`, and `Servers` sub-OUs inside each regional OU to logically separate object types.

<img width="754" height="523" alt="q2Y0jvX - Imgur" src="https://github.com/user-attachments/assets/5a9002b3-8900-4bab-b2cc-12029eb7fead" />



1. Expand your domain — you should now see `USA`, `Europe`, and `Asia`
2. Right-click `USA` → **New → Organizational Unit** → name it `Computers` → click **OK**
3. Repeat to create `Users` and `Servers` under `USA`
4. Repeat the entire process for `Europe` and `Asia`

Separating object types into dedicated sub-OUs allows you to apply different GPOs to computers vs. users, delegate specific admin rights (e.g., help desk can only manage `Users` OUs), and simplify scripting by querying predictable OU paths.

**Result after this step:**

```
computerwiz.com
├── USA
│   ├── Computers
│   ├── Users
│   └── Servers
├── Europe
│   ├── Computers
│   ├── Users
│   └── Servers
└── Asia
    ├── Computers
    ├── Users
    └── Servers
```

---

### Step 3 — Create a Security Group

**Goal:** Create an `IT` Security Group inside `USA > Users` to manage access permissions for IT staff.

<img width="754" height="528" alt="8fF4Tzq - Imgur" src="https://github.com/user-attachments/assets/04534315-5276-4b02-bd2b-9ef5ea369930" />


<img width="752" height="526" alt="2Ozh2G9 - Imgur" src="https://github.com/user-attachments/assets/45dff37e-5219-4015-8d18-bc5225b85d1d" />


1. In the left pane, expand **USA** and click on **Users**
2. Right-click in the right pane → **New → Group**
3. Fill in the dialog:

| Field       | Value        |
| ----------- | ------------ |
| Group name  | `IT`         |
| Group scope | **Global**   |
| Group type  | **Security** |

4. Click **OK**

Security Groups are used to assign permissions to resources (shared folders, printers, applications). Global scope means the group can contain accounts from the same domain and be used in any domain in the forest.

---

### Step 4 — Create a Distribution Group

**Goal:** Create a `DL-IT Admins` Distribution Group inside `USA > Users` for email communications to all IT administrators.

<img width="750" height="519" alt="cMfomJw - Imgur" src="https://github.com/user-attachments/assets/f32d72e9-1bb3-4c15-8f6d-488261066ad0" />

1. Navigate to **USA → Users**
2. Right-click → **New → Group**
3. Fill in the dialog:

| Field       | Value            |
| ----------- | ---------------- |
| Group name  | `DL-IT Admins`   |
| Group scope | **Global**       |
| Group type  | **Distribution** |

4. Click **OK**

Distribution Groups are email-only — they cannot be used to assign resource permissions. The `DL-` prefix is a naming convention indicating a Distribution List, making it instantly identifiable in a large directory.

**Security vs. Distribution — Quick Reference:**

| Feature                        | Security Group | Distribution Group |
| ------------------------------ | -------------- | ------------------ |
| Assign file/folder permissions | ✅ Yes          | ❌ No               |
| Used as email list             | ✅ Yes          | ✅ Yes              |
| Apply GPOs                     | ✅ Yes          | ❌ No               |
| Typical naming prefix          | `GRP-`, `SG-`  | `DL-`              |

---

### Directory Structure Summary

After completing all steps, your Active Directory structure should look like this:

```
computerwiz.com
├── USA
│   ├── Computers
│   ├── Users
│   │   ├── IT              [Global Security Group]
│   │   └── DL-IT Admins    [Global Distribution Group]
│   └── Servers
├── Europe
│   ├── Computers
│   ├── Users
│   └── Servers
└── Asia
    ├── Computers
    ├── Users
    └── Servers
```

---

### PowerShell Equivalent — Part 1

```powershell
# Create top-level OUs
New-ADOrganizationalUnit -Name "USA"    -Path "DC=computerwiz,DC=com"
New-ADOrganizationalUnit -Name "Europe" -Path "DC=computerwiz,DC=com"
New-ADOrganizationalUnit -Name "Asia"   -Path "DC=computerwiz,DC=com"

# Create sub-OUs for each region
foreach ($region in "USA", "Europe", "Asia") {
    foreach ($sub in "Computers", "Users", "Servers") {
        New-ADOrganizationalUnit -Name $sub -Path "OU=$region,DC=computerwiz,DC=com"
    }
}

# Create Security Group in USA/Users
New-ADGroup -Name "IT" `
            -GroupScope Global `
            -GroupCategory Security `
            -Path "OU=Users,OU=USA,DC=computerwiz,DC=com"

# Create Distribution Group in USA/Users
New-ADGroup -Name "DL-IT Admins" `
            -GroupScope Global `
            -GroupCategory Distribution `
            -Path "OU=Users,OU=USA,DC=computerwiz,DC=com"
```

---

## 🔧 Part 2 — Domain Controller: Users, Groups & Password Management

This lab documents real password reset, user account management, and group membership workflows on `corp.local` — built to mirror enterprise IT helpdesk tasks.

**What this lab covers:**
- Creating and configuring user accounts in Active Directory
- Organizing users into Organizational Units (OUs)
- Creating and managing Security Groups and Distribution Groups
- Performing password resets via GUI and PowerShell
- Unlocking locked accounts
- Disabling and enabling accounts
- Verifying changes from a domain-joined client machine

---

### Part 2 Environment

```
┌──────────────────────────────────────────────────────────────────────┐
│  OS            Windows Server 2022 (Domain Controller)               │
│  CLIENT        Windows 11 Pro (Domain-joined)                        │
│  HYPERVISOR    Oracle VirtualBox — Latest Version                    │
│  TOOLS USED    Active Directory Users & Computers (ADUC)             │
│                Active Directory Administrative Center (ADAC)         │
│                PowerShell (Active Directory Module)                  │
│  FOCUS AREA    Password resets · User management · Group membership  │
└──────────────────────────────────────────────────────────────────────┘
```

---

### Section 1 — OU Structure

#### OU Hierarchy

```
corp.local
├── _CORP
│   ├── IT
│   │   ├── Helpdesk
│   │   └── SysAdmins
│   ├── HR
│   ├── Finance
│   ├── Marketing
│   ├── Sales
│   └── _DISABLED ACCOUNTS
└── _GROUPS
    ├── Security Groups
    └── Distribution Groups
```

#### Creating OUs (GUI)

1. Open **Server Manager → Tools → Active Directory Users and Computers**
2. Right-click the domain root `corp.local`
3. Select **New → Organizational Unit**
4. Name it `_CORP` — check **"Protect container from accidental deletion"**
5. Repeat to create sub-OUs: `IT`, `HR`, `Finance`, `Marketing`, `Sales`
6. Create `_DISABLED ACCOUNTS` OU for offboarded users

#### Creating OUs (PowerShell)

```powershell
# Create top-level OU
New-ADOrganizationalUnit -Name "_CORP" -Path "DC=corp,DC=local" -ProtectedFromAccidentalDeletion $true

# Create department OUs inside _CORP
$departments = @("IT","HR","Finance","Marketing","Sales","_DISABLED ACCOUNTS")
foreach ($dept in $departments) {
    New-ADOrganizationalUnit -Name $dept -Path "OU=_CORP,DC=corp,DC=local" -ProtectedFromAccidentalDeletion $true
}

Write-Host "All OUs created successfully." -ForegroundColor Green
```

---

### Section 2 — User Account Management

#### Creating a New User (GUI)

1. In ADUC, expand `corp.local → _CORP → HR`
2. Right-click the **HR** OU → **New → User**
3. Fill in the form:

| Field           | Example Value        |
| --------------- | -------------------- |
| First Name      | Linda                |
| Last Name       | Torres               |
| Full Name       | Linda Torres         |
| User Logon Name | `ltorres@corp.local` |

4. Click **Next**
5. Set a temporary password: `Welcome@2025!`
6. Check **"User must change password at next logon"**
7. Click **Next → Finish**

#### Creating Users in Bulk (PowerShell)

```powershell
$users = @(
    @{First="Sarah"; Last="Johnson"; Dept="Marketing"; OU="OU=Marketing,OU=_CORP,DC=corp,DC=local"},
    @{First="Marcus"; Last="Lee";     Dept="Sales";     OU="OU=Sales,OU=_CORP,DC=corp,DC=local"},
    @{First="Priya";  Last="Nair";    Dept="Finance";   OU="OU=Finance,OU=_CORP,DC=corp,DC=local"},
    @{First="James";  Last="Carter";  Dept="IT";        OU="OU=IT,OU=_CORP,DC=corp,DC=local"},
    @{First="Linda";  Last="Torres";  Dept="HR";        OU="OU=HR,OU=_CORP,DC=corp,DC=local"}
)

foreach ($u in $users) {
    $username  = ($u.First[0] + $u.Last).ToLower()
    $upn       = "$username@corp.local"
    $fullname  = "$($u.First) $($u.Last)"
    $password  = ConvertTo-SecureString "Welcome@2025!" -AsPlainText -Force

    New-ADUser `
        -GivenName         $u.First `
        -Surname           $u.Last `
        -Name              $fullname `
        -SamAccountName    $username `
        -UserPrincipalName $upn `
        -Path              $u.OU `
        -Department        $u.Dept `
        -AccountPassword   $password `
        -ChangePasswordAtLogon $true `
        -Enabled           $true

    Write-Host "Created: $fullname ($upn)" -ForegroundColor Cyan
}
```

---

### Section 3 — Security Groups & Group Membership

#### Groups Created

| Group Name              | Type         | Scope        | Purpose                              |
| ----------------------- | ------------ | ------------ | ------------------------------------ |
| GRP-IT-Staff            | Security     | Global       | All IT department staff              |
| GRP-HR-Staff            | Security     | Global       | All HR department staff              |
| GRP-Finance-Staff       | Security     | Global       | All Finance department staff         |
| GRP-Marketing-Staff     | Security     | Global       | All Marketing department staff       |
| GRP-Sales-Staff         | Security     | Global       | All Sales department staff           |
| GRP-SharedDrive-Finance | Security     | Domain Local | Access to Finance shared folder      |
| GRP-SharedDrive-HR      | Security     | Domain Local | Access to HR shared folder           |
| DIST-AllStaff           | Distribution | Global       | Company-wide email distribution list |

#### Creating a Security Group (GUI)

1. In ADUC, expand `corp.local → _GROUPS → Security Groups`
2. Right-click → **New → Group**
3. Fill in: Group Name `GRP-Finance-Staff`, Scope `Global`, Type `Security`
4. Click **OK**
5. Double-click the group → **Members tab → Add** → type the username → **Check Names → OK**

#### Adding Users to Groups (PowerShell)

```powershell
Add-ADGroupMember -Identity "GRP-IT-Staff"        -Members "jcarter"
Add-ADGroupMember -Identity "GRP-HR-Staff"        -Members "ltorres"
Add-ADGroupMember -Identity "GRP-Finance-Staff"   -Members "pnair"
Add-ADGroupMember -Identity "GRP-Marketing-Staff" -Members "sjohnson"
Add-ADGroupMember -Identity "GRP-Sales-Staff"     -Members "mlee"

# Verify membership
Get-ADGroupMember -Identity "GRP-Finance-Staff" | Select-Object Name, SamAccountName

# Check all groups a user belongs to
Get-ADPrincipalGroupMembership -Identity "pnair" | Select-Object Name, GroupCategory
```

---

### Section 4 — Password Reset

**Ticket TKT-2001:** User `ltorres` (Linda Torres, HR) cannot log in — forgot her password after returning from leave.

#### Method 1: GUI (ADUC)

1. Open **Active Directory Users and Computers**
2. Navigate to `_CORP → HR`
3. Right-click **Linda Torres → Reset Password**
4. Enter a new temporary password: `TempPass@2025!`
5. Check **"User must change password at next logon"**
6. Check **"Unlock the user's account"**
7. Click **OK** and notify the user via phone or secure message

#### Method 2: PowerShell

```powershell
$newPassword = ConvertTo-SecureString "TempPass@2025!" -AsPlainText -Force

Set-ADAccountPassword -Identity "ltorres" -NewPassword $newPassword -Reset
Set-ADUser -Identity "ltorres" -ChangePasswordAtLogon $true
Unlock-ADAccount -Identity "ltorres"

Get-ADUser -Identity "ltorres" -Properties LockedOut, PasswordLastSet, PasswordExpired |
    Select-Object Name, LockedOut, PasswordLastSet, PasswordExpired

Write-Host "Password reset complete for ltorres." -ForegroundColor Green
```

#### Method 3: Active Directory Administrative Center (ADAC)

1. Open **Server Manager → Tools → Active Directory Administrative Center**
2. Navigate to `corp (local) → _CORP → HR`
3. Double-click **Linda Torres**
4. In the Actions panel, click **Reset password**
5. Enter and confirm the temporary password
6. Check **"User must change password at next logon"** → click **OK**

---

### Section 5 — Account Lockout & Unlock

**Ticket TKT-2002:** User `mlee` (Marcus Lee, Sales) is locked out after too many failed login attempts.

```powershell
# Check lockout status
Get-ADUser -Identity "mlee" -Properties LockedOut, BadLogonCount, LastBadPasswordAttempt |
    Select-Object Name, LockedOut, BadLogonCount, LastBadPasswordAttempt

# Unlock without resetting password
Unlock-ADAccount -Identity "mlee"

# Confirm
Get-ADUser -Identity "mlee" -Properties LockedOut | Select-Object Name, LockedOut
```

**GUI Method:** Right-click user in ADUC → **Properties → Account tab** → check **"Unlock account"** → **Apply → OK**

---

### Section 6 — Disabling & Enabling Accounts

**Ticket TKT-2003:** Employee `sjohnson` (Sarah Johnson, Marketing) has left the company. Disable account per offboarding policy.

```powershell
# Disable and move to disabled accounts OU
Disable-ADAccount -Identity "sjohnson"

Move-ADObject -Identity "CN=Sarah Johnson,OU=Marketing,OU=_CORP,DC=corp,DC=local" `
    -TargetPath "OU=_DISABLED ACCOUNTS,OU=_CORP,DC=corp,DC=local"

# Confirm
Get-ADUser -Identity "sjohnson" -Properties Enabled | Select-Object Name, Enabled

# Re-enable if needed
Enable-ADAccount -Identity "sjohnson"
```

**GUI Method:** Right-click user → **Disable Account**, then right-click → **Move** → select `_DISABLED ACCOUNTS` OU

---

### Section 7 — Verification from Client Machine

After all changes, log in to `WIN-CLIENT01` and verify:

```
1. Log in as ltorres   — confirm forced password change prompt appears
2. Log in as mlee      — confirm successful login after unlock
3. Attempt as sjohnson — confirm "Account disabled" error message
4. Run: gpresult /r    — confirm Group Policy applied from WIN-DC01
```

---

## ✅ Summary of Tasks Completed

| Task                                   | User                    | Method Used            | Ticket   |
| -------------------------------------- | ----------------------- | ---------------------- | -------- |
| Created regional OUs (USA/Europe/Asia) | —                       | GUI + PowerShell       | —        |
| Created Security & Distribution Groups | —                       | GUI + PowerShell       | —        |
| Created 5 user accounts                | Multiple                | PowerShell bulk script | —        |
| Created 8 security/distribution groups | —                       | GUI + PowerShell       | —        |
| Assigned users to department groups    | Multiple                | PowerShell             | —        |
| Password reset                         | ltorres                 | GUI + PowerShell       | TKT-2001 |
| Account unlock                         | mlee                    | PowerShell             | TKT-2002 |
| Account disable + OU move              | sjohnson                | PowerShell             | TKT-2003 |
| Verified changes from client           | ltorres, mlee, sjohnson | Windows 11 login       | —        |

---

## 📖 Key Concepts

| Concept                      | Description                                                                                              |
| ---------------------------- | -------------------------------------------------------------------------------------------------------- |
| **Organizational Unit (OU)** | A container in AD used to organise users, computers, and groups. Enables GPO application and delegation. |
| **Security Group**           | Assigns permissions to shared resources and applies Group Policy.                                        |
| **Distribution Group**       | An email distribution list — cannot be used for security permissions.                                    |
| **Global Scope**             | Group contains members from the same domain; usable in any domain in the forest.                         |
| **Domain Local Scope**       | Used to assign permissions to resources within a single domain.                                          |
| **Universal Scope**          | Contains members from any domain in the forest; for multi-domain environments.                           |
| **Password Reset**           | Can be performed via ADUC, ADAC, or PowerShell (`Set-ADAccountPassword`).                                |
| **Account Lockout**          | Triggered by failed login attempts; resolved with `Unlock-ADAccount` or via ADUC.                        |
| **Account Disable**          | Used during offboarding; disables login access without deleting the account or its data.                 |

---

*Built by [Eric Frimpong](https://github.com/eric-frimpong) — IT Support Specialist | New York, NY*
