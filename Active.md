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

<img width="928" height="933" alt="image" src="https://github.com/user-attachments/assets/88c91800-abde-4a59-9e73-fb80e3b4165b" />



enum4linux

<img width="873" height="533" alt="image" src="https://github.com/user-attachments/assets/755a794c-6674-4b0f-9056-2752925ebf2a" />


# Exploit


## port 135 rpc

rien

<img width="735" height="185" alt="image" src="https://github.com/user-attachments/assets/ec2ec604-91c7-41d8-8c10-4f12b2cd7af4" />




## port 389 ldap


Il faut qu'on bind pour voir les informations.

<img width="846" height="409" alt="image" src="https://github.com/user-attachments/assets/4d8d43ca-751b-4dde-8e18-7e6549adacf6" />



## port 445 smb 

Besoin de credentials.

<img width="632" height="150" alt="image" src="https://github.com/user-attachments/assets/cd238c56-4b32-49de-9d27-b7ab4ad90cc5" />




Je peux avoir une connexion anonyme en ne spécifiant aucun nom d'utilisateur ou mot de passe.

<img width="572" height="214" alt="image" src="https://github.com/user-attachments/assets/646a0a88-d474-4449-9dc2-80077b88fe3a" />



Je télécharge tous les fichiers sur le share.

Il y a group.xml qui contient un nom d'utilisateur avec des credentials.

<img width="835" height="317" alt="image" src="https://github.com/user-attachments/assets/93e238ca-1122-4c0a-a865-1ddbc9a64074" />



Le cpassword peut être décrypté avec gpp-decrypt.

<img width="958" height="169" alt="image" src="https://github.com/user-attachments/assets/31ba01cf-746b-4051-8a60-af11e3c8deb7" />




## retour smb


Avec smbmap, on peut revoir les partages qui sont accessibles pour cet usager.

<img width="952" height="451" alt="image" src="https://github.com/user-attachments/assets/5e14b7fc-ff61-48ae-914a-ae69fccae1be" />




Dans le `/home/kali/HTB/Active/SVC_TGS/Desktop`, je retrouve un user.txt et j'ai obtenu le premier flag.

<img width="522" height="113" alt="image" src="https://github.com/user-attachments/assets/ce9cf59e-7738-4bda-9d01-aca9ff8fbbda" />




# PrivEsc



## kerberoast


J'effectue cette commande pour obtenir un potentiel tgs :
```
sudo impacket-GetUserSPNs -request -dc-ip 10.129.24.11 active.htb/'svc_tgs':GPPstillStandingStrong2k18
```

Et j'obtiens un tgs de l'usager administrator.

<img width="948" height="771" alt="image" src="https://github.com/user-attachments/assets/34ca70a9-1583-4bf0-a647-08418af393c6" />



Je crack le hash à l'aide de hashcat et je trouve son mot de passe.

<img width="1050" height="598" alt="image" src="https://github.com/user-attachments/assets/f4e50f7a-9592-4c37-abc4-61bf8402e45b" />



Je me connecte sur le smb dans le share C$ pour retrouver le flag.

<img width="886" height="195" alt="image" src="https://github.com/user-attachments/assets/ef6be0c9-31c4-4de0-bf01-4b4cfab8e043" />




Je pouvais aussi obtenir un shell avec psexec comme suit.

<img width="859" height="361" alt="image" src="https://github.com/user-attachments/assets/db4a905c-4673-4933-a7c8-8d101358e30c" />

