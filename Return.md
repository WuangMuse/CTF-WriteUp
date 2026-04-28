---
date: 2026-04-28
tags:
  - htb
  - easy
  - AD
  - ldap
  - pass-back
  - group_priv
  - server_operator
---


# Recon




TCP Scan :

<img width="941" height="899" alt="image" src="https://github.com/user-attachments/assets/4be3452c-92b1-4f0a-97c2-30e14bf3679e" />



UDP Scan :

<img width="933" height="528" alt="image" src="https://github.com/user-attachments/assets/91eb43b0-9d1c-4d17-a274-8a5e07782bd2" />




# Exploit


## Port 80

The main page of return.local.

<img width="1149" height="744" alt="image" src="https://github.com/user-attachments/assets/df22cdf1-63be-4d2c-944a-ba264f113755" />



There's a settings section.

<img width="867" height="419" alt="image" src="https://github.com/user-attachments/assets/c0223e14-107a-42eb-b3a5-7129fb2cafaf" />





### Pass-Back Attack

I can do a pass-back attack to catch LDAP credentials by opening a listener.
.
<img width="978" height="309" alt="image" src="https://github.com/user-attachments/assets/2f134ad1-2f8a-4848-a6d7-0e583e626fd0" />

<img width="656" height="107" alt="image" src="https://github.com/user-attachments/assets/43d3e227-a3ac-4342-a44d-a4ae65f17457" />

Here, we redirect the printers authentication to our computer by changing the server address to our IP. The credential is pass in plaintexte to our machine.


## svc-printer

I will check if the credentials allow a connection through WinRM like so.

<img width="958" height="265" alt="image" src="https://github.com/user-attachments/assets/1e5c6731-3af9-4153-872d-6bbd4574ceee" />



After connecting with evil-winrm, I am able to find the flag.

<img width="687" height="377" alt="image" src="https://github.com/user-attachments/assets/a9294253-8b49-4721-8318-c3eee67f5cac" />



# PrivEsc

I find multiple privileges that the user can use.

<img width="766" height="380" alt="image" src="https://github.com/user-attachments/assets/3dd5a733-5575-422c-9daa-8b986a285e2c" />




After trying some privesc methods for the known privileges, I rotate to checking the groups that the user is part of. svc-printer is part of Server Operators, a group that can be abused.

<img width="782" height="601" alt="image" src="https://github.com/user-attachments/assets/38178546-93b4-4535-b4d9-1b1ff85d7b2b" />


The Server Operators group is able to change service configurations. I will change the binpath of VSS to point to nc64.exe, allowing me to obtain a reverse shell. Since VSS runs as SYSTEM, this will give me a SYSTEM-level reverse shell.

I will drop nc64.exe and configure the VSS binpath.

<img width="976" height="101" alt="image" src="https://github.com/user-attachments/assets/5bd76028-af00-44ba-a0cc-8d0936660b05" />




With an open listener, I start VSS, giving me a shell.

<img width="922" height="100" alt="image" src="https://github.com/user-attachments/assets/a12fb257-1742-4a48-b2f7-3258cf6a1d11" />




I am able to find the flag.

<img width="668" height="543" alt="image" src="https://github.com/user-attachments/assets/28984795-48a4-454d-8437-86193ad54b0d" />



