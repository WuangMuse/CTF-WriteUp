---
date: 2026-05-05
tags:
  - htb
  - linux
  - easy
  - gitea
  - searchor
  - docker
  - path_hijack
  - suid
---


# Recon

TCP Scan : 

<img width="942" height="518" alt="image" src="https://github.com/user-attachments/assets/077939e3-9a09-44fb-b396-3560635cf226" />






# Exploit


## port 80

<img width="1193" height="807" alt="image" src="https://github.com/user-attachments/assets/1095ed4f-d1c8-43ff-ab78-6f3879abcd9f" />


<img width="975" height="476" alt="image" src="https://github.com/user-attachments/assets/68a216b1-3be2-49d4-88b1-f747ffa79adc" />



Puisque j’ai la version de searchor, je peux essayer de trouver des exploits sur cette technologie. Je trouve qu’il y a une vulnérabilité avec l’utilisation de eval en Python.


<img width="903" height="652" alt="image" src="https://github.com/user-attachments/assets/efd1eb36-2c90-4042-b554-9cdbd79cd24d" />



Je trouve ce script :
https://github.com/libertycityhacker/CVE-2023-43364-Exploit-CVE/blob/main/exploit.py

Je le run et j’obtiens une connexion sur mon listener.

<img width="1005" height="261" alt="image" src="https://github.com/user-attachments/assets/a612c4af-1680-4837-bd41-bf7239a0ffca" />



Je trouve le premier flag dans le home du user.


# PrivEsc


## apache 

Je retrouve qu’il y a un sous-domaine pour gitea.

<img width="671" height="620" alt="image" src="https://github.com/user-attachments/assets/aec3a98e-a6b9-442d-aebd-a6a3622b25bf" />



## gitea

<img width="1243" height="747" alt="image" src="https://github.com/user-attachments/assets/8aa0a900-5e9c-42ce-9513-dc90f63827ae" />



J'ai trouvé le mot de passe de cody.

<img width="919" height="284" alt="image" src="https://github.com/user-attachments/assets/ba524f77-a27c-49be-b212-906736011793" />


Rien de trouvé sur le site Gitea.


## suid



Le mot de passe :


<img width="940" height="262" alt="image" src="https://github.com/user-attachments/assets/032b4c67-8f61-46af-9f62-aa7d6eef7a04" />


<img width="811" height="192" alt="image" src="https://github.com/user-attachments/assets/e3a527c6-bd31-4e58-8b86-5f42eb22fcef" />



Si je fais ça :

`sudo python3 /opt/scripts/system-checkup.py docker-inspect '{{json .}}' gitea | jq .`



<img width="1338" height="822" alt="image" src="https://github.com/user-attachments/assets/4c1f6908-2f47-4c10-a414-c8a71b159efa" />



Je trouve un mdp.

<img width="849" height="431" alt="image" src="https://github.com/user-attachments/assets/ff896b2d-458a-4408-bb3f-8a77ee789eb3" />



Je suis capable de se connecter à administrator sur Gitea avec le mdp.

Dans le script system-checkup.py, on retrouve que ça appelle ce script :

<img width="424" height="266" alt="image" src="https://github.com/user-attachments/assets/d8dff742-49ed-4aa1-9992-07fc47b24dd8" />




Si on run full-checkup :

<img width="1350" height="872" alt="image" src="https://github.com/user-attachments/assets/4ad473ca-a6d8-4014-9049-b633390d5d01" />




Ce qu’on peut faire, c’est créer un nouveau full-checkup.sh qui est un reverse shell. Lorsqu’on run le paramètre full-checkup.sh, ça run le script dans le folder actuel.

<img width="937" height="108" alt="image" src="https://github.com/user-attachments/assets/dfecea1c-1f95-4ba4-b8e8-f3d4da517dd2" />


<img width="775" height="76" alt="image" src="https://github.com/user-attachments/assets/28307c49-f4d4-4b67-bca2-cc2f19ac0462" />

<img width="846" height="266" alt="image" src="https://github.com/user-attachments/assets/5024321e-8db1-48b3-b88e-5dc7b7f89495" />

