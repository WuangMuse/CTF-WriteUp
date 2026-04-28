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

![[Pasted image 20260428101428.png]]


UDP Scan :

![[Pasted image 20260428101529.png]]





# Exploit


## Port 80

The main page of return.local.

![[Pasted image 20260428101906.png]]


There's a settings section.

![[Pasted image 20260428102226.png]]




### Pass-Back Attack

I can do a pass-back attack to catch LDAP credentials by opening a listener.
.
![[Pasted image 20260428111904.png]]

![[Pasted image 20260428111857.png]]

Here, we redirect the printers authentication to our computer by changing the server address to our IP. The credential is pass in plaintexte to our machine.


## svc-printer

I will check if the credentials allow a connection through WinRM like so.

![[Pasted image 20260428163406.png]]


After connecting with evil-winrm, I am able to find the flag.

![[Pasted image 20260428163352.png]]


# PrivEsc

I find multiple privileges that the user can use.

![[Pasted image 20260428163738.png]]


After trying some privesc methods for the known privileges, I rotate to checking the groups that the user is part of. svc-printer is part of Server Operators, a group that can be abused.

![[Pasted image 20260428171602.png]]

The Server Operators group is able to change service configurations. I will change the binpath of VSS to point to nc64.exe, allowing me to obtain a reverse shell. Since VSS runs as SYSTEM, this will give me a SYSTEM-level reverse shell.

I will drop nc64.exe and configure the VSS binpath.

![[Pasted image 20260428172931.png]]


With an open listener, I start VSS, giving me a shell.

![[Pasted image 20260428173017.png]]


I am able to find the flag.

![[Pasted image 20260428173030.png]]


