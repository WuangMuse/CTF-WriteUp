---
date: 2026-04-22
tags:
  - htb
  - easy
  - AD
  - asrep-roast
  - cached_creds
  - dcsync
  - secretdump
---

# Recon


TCP scan:

![[Pasted image 20260422185632.png]]

I see the domain name.


# Exploit


## port 389 LDAP


I will check the LDAP with this command : `ldapsearch -x -H ldap://EGOTISTICAL-BANK.LOCAL -b "dc=EGOTISTICAL-BANK,dc=LOCAL"`. It outputs a lot of information.

![[Pasted image 20260422190007.png]]

There's not a lot of things that are interesting except the username Hugo Smith.


![[Pasted image 20260422190131.png]]



## port 80


On the web page, we see this : 

![[Pasted image 20260422190550.png]]


I see a list of users that I'll add to my users.txt.

![[Pasted image 20260422190753.png]]


## port 88 kerberos

I had a lot of trouble finding a way in. I can enumerate users with kerbrute, and I found some users.

![[Pasted image 20260422191746.png]]


## AS-REP roast


With the new users that I found, I am able to find a hash with AS-REP. I ran this command : `impacket-GetNPUsers EGOTISTICAL-BANK.LOCAL/ -dc-ip 10.129.24.159 -no-pass -usersfile users.txt`

![[Pasted image 20260422192016.png]]


I crack it with hashcat and I find the password.

![[Pasted image 20260422192123.png]]


### passwordspray

I decide to password spray with my user list in case it's fruitful, so I can have access to other users. In this case, it didn't work and the password only works for fsmith.

![[Pasted image 20260422192347.png]]


## fsmith


I connect to the user fsmith and found the first flag.

![[Pasted image 20260422192455.png]]




# PrivEsc



## Bloodhound 


I'll use bloodhound like this : 
`bloodhound-python -u 'fsmith' -p 'Thestrokes23' -d EGOTISTICAL-BANK.LOCAL -c All -ns 10.129.24.159`

I am able to find that the user HSmith can have the hash exposed by doing a kerberoast, but it doesn't work


![[Pasted image 20260422200156.png]]



## WinPeas

I am out of ideas so I'll run winpeas.

Found the password of svc_loanmanager.

![[Pasted image 20260422200509.png]]


## DSync 

Per bloodhound, the new user found can perform a DCSync attack.

![[Pasted image 20260422200702.png]]


I am not able to log in to that user.


![[Pasted image 20260422201410.png]]


### Back on fsmith

that the user svc_loanmanager is not on the computer but there's one that resembles it named svc_loanmgr.

![[Pasted image 20260422201554.png]]


The password found earlier works for this user.

![[Pasted image 20260422201630.png]]



### svc_loanmgr

I can now run dcsync attack on the domain with svc_loanmgr.

![[Pasted image 20260422201733.png]]


## Administrator

I connect to the administrator user with pass-the-hash on evil-winrm and I get the flag.

![[Pasted image 20260422201845.png]]
