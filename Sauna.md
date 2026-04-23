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

<img width="954" height="715" alt="image" src="https://github.com/user-attachments/assets/99c07ac5-499a-4463-9a25-67b2056710c4" />


I see the domain name.


# Exploit


## port 389 LDAP


I will check the LDAP with this command : `ldapsearch -x -H ldap://EGOTISTICAL-BANK.LOCAL -b "dc=EGOTISTICAL-BANK,dc=LOCAL"`. It outputs a lot of information.

<img width="923" height="847" alt="image" src="https://github.com/user-attachments/assets/9cc4738a-6201-4314-b92c-a900c32f3d54" />


There's not a lot of things that are interesting except the username Hugo Smith.


<img width="847" height="442" alt="image" src="https://github.com/user-attachments/assets/5a97c674-448e-4a25-a61a-563135bfee27" />




## port 80


On the web page, we see this : 

<img width="1132" height="623" alt="image" src="https://github.com/user-attachments/assets/4d215b16-2617-483d-b2de-4d46e036fef9" />



I see a list of users that I'll add to my users.txt.

<img width="1071" height="611" alt="image" src="https://github.com/user-attachments/assets/9610dad1-f647-45be-a763-bdaba60e0613" />


## port 88 kerberos

I had a lot of trouble finding a way in. I can enumerate users with kerbrute, and I found some users.

<img width="981" height="404" alt="image" src="https://github.com/user-attachments/assets/7ffa45af-45a5-4f1c-87de-460d60084786" />



## AS-REP roast


With the new users that I found, I am able to find a hash with AS-REP. I ran this command : `impacket-GetNPUsers EGOTISTICAL-BANK.LOCAL/ -dc-ip 10.129.24.159 -no-pass -usersfile users.txt`

<img width="919" height="467" alt="image" src="https://github.com/user-attachments/assets/ce255396-2334-46b6-b0a5-e6e3b7b57471" />


I crack it with hashcat and I find the password.

<img width="950" height="667" alt="image" src="https://github.com/user-attachments/assets/6ee97834-b7de-48d5-9f1c-138b9ac30eb1" />



### passwordspray

I decide to password spray with my user list in case it's fruitful, so I can have access to other users. In this case, it didn't work and the password only works for fsmith.

<img width="940" height="306" alt="image" src="https://github.com/user-attachments/assets/4e7258c2-5157-41be-a53b-860973c3a64d" />



## fsmith


I connect to the user fsmith and found the first flag.

<img width="555" height="384" alt="image" src="https://github.com/user-attachments/assets/892eba38-2d8c-4541-a1f8-6feffdac6603" />




# PrivEsc



## Bloodhound 


I'll use bloodhound like this : 
`bloodhound-python -u 'fsmith' -p 'Thestrokes23' -d EGOTISTICAL-BANK.LOCAL -c All -ns 10.129.24.159`

I am able to find that the user HSmith can have the hash exposed by doing a kerberoast, but it doesn't work


<img width="1085" height="408" alt="image" src="https://github.com/user-attachments/assets/4688076f-63da-4edb-9abf-574297a5ecaa" />




## WinPeas

I am out of ideas so I'll run winpeas.

Found the password of svc_loanmanager.

<img width="852" height="244" alt="image" src="https://github.com/user-attachments/assets/d89cd12a-4f37-4ae4-ac52-96817c9fa3de" />



## DSync 

Per bloodhound, the new user found can perform a DCSync attack.

![[Pasted image 20260422200702.png]]


I am not able to log in to that user.


<img width="1026" height="346" alt="image" src="https://github.com/user-attachments/assets/bb45d8dc-0cb2-4125-b494-33f24d74775a" />



### Back on fsmith

that the user svc_loanmanager is not on the computer but there's one that resembles it named svc_loanmgr.

<img width="937" height="273" alt="image" src="https://github.com/user-attachments/assets/10ebfbe5-a455-4c9b-b093-b36fd5d73efc" />



The password found earlier works for this user.

<img width="740" height="231" alt="image" src="https://github.com/user-attachments/assets/b2f0eeef-6987-4100-8cc4-732e884a34e2" />




### svc_loanmgr

I can now run dcsync attack on the domain with svc_loanmgr.

<img width="943" height="899" alt="image" src="https://github.com/user-attachments/assets/3c744f6a-c052-4ef7-9abd-5c4091af6dec" />



## Administrator

I connect to the administrator user with pass-the-hash on evil-winrm and I get the flag.

<img width="774" height="554" alt="image" src="https://github.com/user-attachments/assets/3ed12ddb-a39a-4b11-96e7-8bf0c4e2dd62" />

