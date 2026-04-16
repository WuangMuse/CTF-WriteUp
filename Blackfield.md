---
date: 2026-04-16
tags:
  - htb
  - hard
  - AD
  - asrep-roast
  - shadow_credential
  - sebackupprivilege
---


# Recon

tcp scan
![[Pasted image 20260416143716.png]]


udp scan
![[Pasted image 20260416144002.png]]


# Exploit


## port 135 rpc

Je tente une connexion avec un null session (usager vide), mais je ne peux rien interroger.

![[Pasted image 20260416144307.png]]


## port 389 ldap

Il en est de même pour ldap.

![[Pasted image 20260416144814.png]]



## port 88 kerberos

Il n'y a pas grand-chose à faire avec les derniers ports ouverts. Je tente donc, comme dernier recours, de brute-forcer une liste d'utilisateurs. Je parviens à récupérer quelques noms.

![[Pasted image 20260416145149.png]]


Jai trouvé comme usager : support, guest et administrator





## as-rep 

Voici ton rapport corrigé. J'ai rectifié l'orthographe et clarifié certains points techniques (comme l'AS-REP Roasting et le fonctionnement du NTDS.dit) tout en gardant ton style.

---

## Recon

**TCP scan** `[Pasted image 20260416143716.png]`

**UDP scan** `[Pasted image 20260416144002.png]`

## Exploit

### Port 135 (RPC)

Je tente une connexion avec un **null session** (usager vide), mais je ne peux rien interroger.

`[Pasted image 20260416144307.png]`

### Port 389 (LDAP)

Il en est de même pour LDAP.

`[Pasted image 20260416144814.png]`

### Port 88 (Kerberos)

Il n'y a pas grand-chose à faire avec les derniers ports ouverts. Je tente donc, comme dernier recours, de **brute-forcer** une liste d'utilisateurs. Je parviens à récupérer quelques noms.

`[Pasted image 20260416145149.png]`

J'ai trouvé les usagers : `support`, `guest` et `administrator`.

### AS-REP Roasting

Puisque j'ai une liste d'utilisateurs, je peux vérifier si le KDC est mal configuré. Cela permet d'obtenir un AS-REP sans mot de passe et d'en extraire le hash pour tenter de le cracker. J'utilise la commande:

`impacket-GetNPUsers BLACKFIELD.local/ -dc-ip 10.129.229.17 -no-pass -usersfile users.txt`

![[Pasted image 20260416150913.png]]


On est capable de le crack avec hashcat et le mdp est `#00^BlackKnight`

## retour sur rpc

De retour sur le port RPC, on peut maintenant interroger le domaine avec les **credentials** que l'on vient d'obtenir.

![[Pasted image 20260416152546.png]]

J'obtiens quelques autres usagers

![[Pasted image 20260416152647.png]]


## port 445 smb

Je regarde les shares accessibles via SMB avec ces identifiants, mais on n'y trouve rien d'exploitable.

![[Pasted image 20260416152935.png]]



![[Pasted image 20260416153121.png]]





## bloodhound

J'utilise BloodHound pour déterminer s'il existe des chemins d'attaque (attack paths) possibles.

![[Pasted image 20260416153738.png]]




### forcechangepassword


On voit que l'usager que je contrôle détient le privilège ForceChangePassword sur un autre compte.

![[Pasted image 20260416154523.png]]

J'essaie la méthode montrée dans BloodHound, mais elle ne fonctionne pas directement.

![[Pasted image 20260416154712.png]]


Par contre, on peut le faire avec une connection rpc. 
![[Pasted image 20260416155001.png]]

Même si la sortie n'indique pas explicitement que cela a réussi, le nouveau mot de passe fonctionne bel et bien.

![[Pasted image 20260416154942.png]]


## smb avec audit 2020

nsuite, je regarde si l'usager **audit2020** a des accès spéciaux. Il possède les droits de lecture sur le partage forensic.
`smbmap -H 10.129.229.17  -u audit2020 -p 'newP@ssword2022'`

![[Pasted image 20260416155047.png]]

On y retrouve des choses interessantes que je telecharge.

![[Pasted image 20260416155206.png]]


Je récupère un dump de LSASS (Local Security Authority Subsystem Service). Ce processus contient les hashs et parfois les mots de passe des usagers connectés. J'extrais le zip pour obtenir le .DMP. 

![[Pasted image 20260416161258.png]]

### pypykatz

Pour lire le fichier .DMP, je dois installer pypykatz.

![[Pasted image 20260416161310.png]]

'exécute l'application et j'y retrouve les hashs pour svc_backup et administrator.

![[Pasted image 20260416161404.png]]



```
svc_backup / 9658d1d1dcd9250115e2205d9f48400d
administrator / 7f1e4ff8c6a8e6b6fcae2d9c0572cd62
```


### hash spray

Je tente un pass-the-hash (ou password spray avec les hashs) sur le domaine. Seul celui de svc_backup marche.

![[Pasted image 20260416162249.png]]

![[Pasted image 20260416162318.png]]


![[Pasted image 20260416162426.png]]

Je me connecte avec cette session et je récupère le flag.

![[Pasted image 20260416162442.png]]



# PrivEsc

On voit que l'usager détient le privilège SeBackupPrivilege. Cela permet de lire n'importe quel fichier, même s'il est verrouillé par le système, en créant une Volume Shadow Copy.

![[Pasted image 20260416162830.png]]


## seBackupPrivilege avec shadow copy


Je me base sur ca : https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/

Je crée un fichier de script `shadow.txt` pour l'outil `diskshadow`.

![[Pasted image 20260416164121.png]]

Il faut le convertir a un format DOS.

![[Pasted image 20260416164208.png]]

Je transfère le fichier sur la cible et j'exécute diskshadow.

![[Pasted image 20260416164341.png]]

![[Pasted image 20260416164350.png]]

Cela crée une copie du fichier ntds.dit, qui contient tous les hashs des comptes du domaine.

![[Pasted image 20260416164621.png]]

J'ai également besoin du fichier SYSTEM (ruche du registre) qui contient la clé de démarrage permettant de décrypter le contenu de `ntds.dit`.

![[Pasted image 20260416164744.png]]


Je telecharge les 2 fichiers.

![[Pasted image 20260416165154.png]]


Ensuite, j'utilise secretsdump pour extraire les hashs localement

![[Pasted image 20260416165252.png]]

J'obtiens ainsi le hash de l'administrateur, ce qui me permet de récupérer le dernier flag.

![[Pasted image 20260416165412.png]]



