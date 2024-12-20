---
title: Recreating Cicada.htb
description: Let’s set up a fully functional Active Directory domain called Cicada.htb on Windows Server 2022!
author: mashhood
date: 2024-12-20 11:32:00 +0800
categories: [System Security]
tags: [htb]
media_subpath: '/assets/img/posts/20241220/'
image:
  path: cicada_recreate.webp
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---

Let’s set up a fully functional Active Directory domain called Cicada.htb on Windows Server 2022! This guide will take you through every step of the process.. If you’re more into automating tasks, check out the PowerShell script at the end. Need help installing Windows Server? You’ll find everything you need in [Installing Windows Server](https://cyberhubs.live/posts/Installing-Windows-Server-2022-in-VMware).

---

### 1. Renaming the PC

Renaming the server with a meaningful name makes it easier to identify in your network, especially in larger or more complex environments.
1. Open **Settings** and click on **View your PC Name**.

![Desktop View](cicada_recreate(1).webp)

2. In the **View your PC Name**, click on **Rename your PC**. Change the Name to **PC-1** and click **Next**.

![Desktop View](cicada_recreate(2).webp)

3. The PC will Restart and Your change will be applied.

---

### 2. Install Active Directory Domain Services (AD DS)
Active Directory Domain Services is the backbone of domain-based networks, enabling centralized management of users, computers, and resources.
1. Open **Server Manager** and click on **Add roles and features**.

![Desktop View](cicada_recreate(3).webp)

2. In the **Add Roles and Features Wizard**, select **Role-based or feature-based installation** and click **Next**.
3. Check the **Active Directory Domain Services** role. When prompted to add features, click **Add Features**.

![Desktop View](cicada_recreate(4).webp)

4. Click **Next** through the Features and AD DS pages, and then click **Install**.

---

### 3. Promote the Server to a Domain Controller
Promoting the server to a domain controller establishes it as the central authority for managing authentication and enforcing security policies.
1. After installation, a notification will appear in Server Manager. Click **Promote this server to a domain controller**.

![Desktop View](cicada_recreate(5).webp)

2. In the **Deployment Configuration** screen:
    - Select **Add a new forest**.
    - Enter the **Root domain name** as `cicada.htb` and click **Next**.

![Desktop View](cicada_recreate(6).webp)

3. In the **Domain Controller Options** screen:
    - Ensure **Domain Name System (DNS)** and **Global Catalog (GC)** are selected.
    - Set the **Directory Services Restore Mode (DSRM)** password.

![Desktop View](cicada_recreate(7).webp)

4. Continue through the wizard, verifying settings and prerequisites, and click **Install**.

The server will restart automatically once the promotion is complete.

---
### 4. Install Certificate Services and Promote the Server
Installing Certificate Services provides the infrastructure for secure communication and identity verification in your domain.

1. Open **Server Manager** and click **Add roles and features** again.
2. In the **Add Roles and Features Wizard**, Check the **Active Directory Certificate Services** role. When prompted, click **Add Features**.

![Desktop View](cicada_recreate(8).webp)

3. Continue through the wizard and click **Install**.
4. After installation, click **Configure Active Directory Certificate Services** on the completion page.

![Desktop View](cicada_recreate(9).webp)

5. In the **AD CS Configuration Wizard**, select the **Certification Authority** role, proceed through the wizard by clicking **Next**, and then click **Configure**.

![Desktop View](cicada_recreate(10).webp)

The server is now set up as a Certificate Authority (CA).

---
### 5. Create Users

After the domain is set up, you can create users in **Active Directory Users and Computers (ADUC)**.

1. Open **Active Directory Users and Computers** from the **Tools** menu in Server Manager.

![Desktop View](cicada_recreate(11).webp)

2. Navigate to **cicada.htb > Users**, right-click, and select **New > User**.

![Desktop View](cicada_recreate(15).webp)

3. Fill in the details for the first user:
    - **Full Name**: Michael Wrightson
    - **User logon name**: michael.wrightson
    - Set a password as `Cicada$M6Corpb*@Lp#nZp!8` and ensure **Password never expires** is checked.
    - Click **Finish**.

![Desktop View](cicada_recreate(16).webp)

![Desktop View](cicada_recreate(17).webp)
Repeat this process for other users using the following details:

| Name           | Logon Name     | Password           |
| -------------- | -------------- | ------------------ |
| David Orelious | david.orelious | `aRt$Lp#7t*VQ!3`   |
| Emily Oscars   | emily.oscars   | `Q!3@Lp#M6b*7t*Vt` |
| John Smoulder  | john.smoulder  | `deROm67F7N^b)=VU` |
| Sarah Dantelia | sarah.dantelia | `XAH1V98-b#F4A!Ux` |

![Desktop View](cicada_recreate(18).webp)

4. After creating the users manually, double-click on David Orelious's account and add `Password is aRt$Lp#7t*VQ!3` to the description field.

![Desktop View](cicada_recreate(31).webp)

---

### 6. Enable and Configure the Guest Account

The **Guest** account is disabled by default. It provides temporary or anonymous access to domain resources. While it can be useful, leaving it enabled with default settings or excessive permissions makes it a prime target for attackers.
Enable it and configure its permissions:
1. Right-click the `Guest` account, select **Properties**, and check **Unlock the Account**.

![Desktop View](cicada_recreate(19).webp)

2. Run the following PowerShell command to enable the account:

```powershell
Enable-ADAccount -Identity Guest
```

![Desktop View](cicada_recreate(20).webp)

---

### 7. Add Emily Oscars to Backup Operators and Remote Management Users

Assigning Emily Oscars to the **Backup Operators** group allows her to perform critical tasks like creating and managing system backups. Additionally, adding her to the **Remote Management Users** group enables her to remotely manage the server, which can be helpful for administrative tasks. However, both groups grant elevated privileges that can be abused if her account is compromised.

To add a user to the **Backup Operators** group:

1. Navigate to **cicada.htb > Users**, locate the **Backup Operators** group, and double-click it.

![Desktop View](cicada_recreate(21).webp)

2. Click **Add**, enter `emily.oscars`, and click **OK**.

![Desktop View](cicada_recreate(22).webp)

3. Repeat the process for the **Remote Management Users** group.

![Desktop View](cicada_recreate(23).webp)

---

### 8. Create SMB Shares

Creating SMB shares centralizes file access and simplifies resource management, but improperly configured shares can expose sensitive files to unauthorized users.

To create file shares for different departments:

1. Open **File Explorer** and create the following directories:
    - `C:\Shares\HR`
    - `C:\Shares\DEV`

![Desktop View](cicada_recreate(25).webp)

2. Open **Server Manager**, go to **File and Storage Services**, and click **Shares**.
3. Right-click and select **New Share**.

![Desktop View](cicada_recreate(26).webp)

4. Select **SMB Share – Quick** and click **Next**.
5. Choose the path (`C:\Shares\HR` for the HR share) and configure permissions as required.

![Desktop View](cicada_recreate(27).webp)

6. Grant full access to `Everyone` for HR.

![Desktop View](cicada_recreate(29).webp)

7. Repeat the process for the `DEV` share, granting access only to `david.orelious` and `emily.oscars`.

![Desktop View](cicada_recreate(30).webp)

---
### PowerShell Script

This PowerShell script automates the setup and configuration of an Active Directory domain in two parts. The first part installs AD DS and sets up the domain, requiring a server restart. The second part, executed post-restart, automates user creation, group assignments, certificate installation, and SMB share configuration. Don't forget to rename the Server with
```powershell
Rename-Computer -NewName "PC-1" -Restart
```

```powershell
Install-WindowsFeature -Name AD-Domain-Services
Install-ADDSForest `
-DomainName "cicada.htb" `
-DomainNetbiosName "CICADA" `
-SafeModeAdministratorPassword (ConvertTo-SecureString "StrongAdminPassword123!" -AsPlainText -Force)
```

```powershell
Import-Module ActiveDirectory

$users = @(
    @{Name="Michael Wrightson"; SamAccountName="michael.wrightson"; Password='Cicada$M6Corpb*@Lp#nZp!8'},
    @{Name="David Orelious"; SamAccountName="david.orelious"; Password='aRt$Lp#7t*VQ!3'},
    @{Name="Emily Oscars"; SamAccountName="emily.oscars"; Password="Q!3@Lp#M6b*7t*Vt"},
    @{Name="John Smoulder"; SamAccountName="john.smoulder"; Password="deROm67F7N^b)=VU"},
    @{Name="Sarah Dantelia"; SamAccountName="sarah.dantelia"; Password="XAH1V98-b#F4A!Ux"}
)

foreach ($user in $users) {
    New-ADUser `
    -Name $user.Name `
    -SamAccountName $user.SamAccountName `
    -UserPrincipalName "$($user.SamAccountName)@cicada.htb" `
    -AccountPassword (ConvertTo-SecureString $user.Password -AsPlainText -Force) `
    -Enabled $true `
    -PasswordNeverExpires $true `
    -Description ($(if ($user.SamAccountName -eq "david.orelious") { "Password is aRt$Lp#7t*VQ!3" } else { "" }))
}

# Add Emily Oscars to Backup Operators and Remote Management Users
Add-ADGroupMember -Identity "Backup Operators" -Members "emily.oscars"
Add-ADGroupMember -Identity "Remote Management Users" -Members "emily.oscars"


# Install AD Certificate Services Role if not installed
Install-WindowsFeature -Name ADCS-Cert-Authority -IncludeManagementTools

# Configure Certificate Services
Install-AdcsCertificationAuthority `
-CAType EnterpriseRootCA `
-CACommonName "CICADA Root CA" `
-KeyLength 2048 `
-HashAlgorithmName SHA256 `
-DatabaseDirectory "C:\Windows\System32\CertLog" `
-LogDirectory "C:\Windows\System32\CertLog"

# Create SMB Shares
New-Item -Path C:\Shares\HR -ItemType Directory
New-Item -Path C:\Shares\DEV -ItemType Directory

# Create Share Permissions
New-SmbShare -Name "HR" -Path "C:\Shares\HR" -FullAccess "Everyone"
icacls "C:\Shares\HR" /grant "Everyone:(F)" /T /C

New-SmbShare -Name "DEV" -Path "C:\Shares\DEV" -FullAccess "cicada\david.orelious", "cicada\emily.oscars"
icacls "C:\Shares\DEV" /grant "cicada\david.orelious:(F)" "cicada\emily.oscars:(F)" /T /C


# Create HR Notice File
@"
Dear new hire!

Welcome to Cicada Corp! We're thrilled to have you join our team. As part of our security protocols, it's essential that you change your default password to something unique and secure.

Your default password is: Cicada$M6Corpb*@Lp#nZp!8

To change your password:

1. Log in to your Cicada Corp account** using the provided username and the default password mentioned above.
2. Once logged in, navigate to your account settings or profile settings section.
3. Look for the option to change your password. This will be labeled as "Change Password".
4. Follow the prompts to create a new password**. Make sure your new password is strong, containing a mix of uppercase letters, lowercase letters, numbers, and special characters.
5. After changing your password, make sure to save your changes.

Remember, your password is a crucial aspect of keeping your account secure. Please do not share your password with anyone, and ensure you use a complex password.

If you encounter any issues or need assistance with changing your password, don't hesitate to reach out to our support team at support@cicada.htb.

Thank you for your attention to this matter, and once again, welcome to the Cicada Corp team!

Best regards,
Cicada Corp
"@ | Out-File -FilePath "C:\Shares\HR\Notice from HR.txt"

# Create Backup Script
@"
$sourceDirectory = "C:\Shares"
$destinationDirectory = "D:\Backup"

$username = "emily.oscars"
$password = ConvertTo-SecureString "Q!3@Lp#M6b*7t*Vt" -AsPlainText -Force
$credentials = New-Object System.Management.Automation.PSCredential($username, $password)
$dateStamp = Get-Date -Format "yyyyMMdd_HHmmss"
$backupFileName = "smb_backup_$dateStamp.zip"
$backupFilePath = Join-Path -Path $destinationDirectory -ChildPath $backupFileName
Compress-Archive -Path $sourceDirectory -DestinationPath $backupFilePath
Write-Host "Backup completed successfully. Backup file saved to: $backupFilePath"
"@ | Out-File -FilePath "C:\Shares\DEV\Backup_script.ps1"


# Unlock the Guest account
Unlock-ADAccount -Identity Guest
# Enable the Guest account
Enable-ADAccount -Identity Guest
```

---

### Conclusion

By following these steps, you can successfully set up the Cicada.htb domain using Windows Server 2022 and Server Manager. This domain is now ready for testing.