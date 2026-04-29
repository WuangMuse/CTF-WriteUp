---
date: 2025-12-05
tags:
  - AD
  - easy
---

# Recon

![[Pasted image 20251205151300.png]]


# Exploit


## smb 
![[Pasted image 20251205151901.png]]



on trouve ca dans le folder 
![[Pasted image 20251205152033.png]]


ds helpdesk
![[Pasted image 20251205152244.png]]

on dezip et on utilise zip2john pr avoir hash et le crack
![[Pasted image 20251205152333.png]]


ensuite, pour extract la cle privce et le certificat (cle public), faut crakc mdp

```
pfx2john.py legacyy_dev_auth.pfx | tee legacyy_dev_auth.pfx.hash
```



![[Pasted image 20251205153022.png]]



easy log in
![[Pasted image 20251205153052.png]]







# PrivEsc



ds psreadline, on retrouive ca 

```
*Evil-WinRM* PS C:\Users\legacyy\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine> ls
```
```
ConsoleHost_history.txt
```

on a creds pr connecterr a lui
```
oxdf@hacky$ evil-winrm -i timelapse.htb -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV' -S
```

ds le compre on a ca
![[Pasted image 20251205153400.png]]

donc on peut faire ca pr get le pass
![[Pasted image 20251205153458.png]]

