---
date: 2026-04-21
tags:
  - htb
  - easy
  - windows
  - AD
  - smb
  - kerberoast
  - cached_creds
---

# Recon

TCP scan : 

![[Pasted image 20260421160341.png]]


enum4linux

![[Pasted image 20260421160431.png]]


# Exploit


## port 135 rpc

rien

![[Pasted image 20260421160512.png]]



## port 389 ldap


Il faut qu'on bind pour voir les informations.

![[Pasted image 20260421160619.png]]


## port 445 smb 

Besoin de credentials.

![[Pasted image 20260421160749.png]]



Je peux avoir une connexion anonyme en ne spécifiant aucun nom d'utilisateur ou mot de passe.

![[Pasted image 20260421163129.png]]


Je télécharge tous les fichiers sur le share.

Il y a group.xml qui contient un nom d'utilisateur avec des credentials.

![[Pasted image 20260421163401.png]]


Le cpassword peut être décrypté avec gpp-decrypt.

![[Pasted image 20260421163729.png]]




## retour smb


Avec smbmap, on peut revoir les partages qui sont accessibles pour cet usager.

![[Pasted image 20260421163847.png]]



Dans le `/home/kali/HTB/Active/SVC_TGS/Desktop`, je retrouve un user.txt et j'ai obtenu le premier flag.

![[Pasted image 20260421164114.png]]




# PrivEsc



## kerberoast


J'effectue cette commande pour obtenir un potentiel tgs :
```
sudo impacket-GetUserSPNs -request -dc-ip 10.129.24.11 active.htb/'svc_tgs':GPPstillStandingStrong2k18
```

Et j'obtiens un tgs de l'usager administrator.

![[Pasted image 20260421164752.png]]


Je crack le hash à l'aide de hashcat et je trouve son mot de passe.

![[Pasted image 20260421164925.png]]


Je me connecte sur le smb dans le share C$ pour retrouver le flag.

![[Pasted image 20260421165546.png]]



Je pouvais aussi obtenir un shell avec psexec comme suit.

![[Pasted image 20260421165921.png]]
