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

![[Pasted image 20260505123915.png]]





# Exploit


## port 80

![[Pasted image 20260505123911.png]]

![[Pasted image 20260505124428.png]]


Puisque j’ai la version de searchor, je peux essayer de trouver des exploits sur cette technologie. Je trouve qu’il y a une vulnérabilité avec l’utilisation de eval en Python.


![[Pasted image 20260505124418.png]]



Je trouve ce script :
https://github.com/libertycityhacker/CVE-2023-43364-Exploit-CVE/blob/main/exploit.py

Je le run et j’obtiens une connexion sur mon listener.

![[Pasted image 20260505125315.png]]


Je trouve le premier flag dans le home du user.


# PrivEsc


## apache 

Je retrouve qu’il y a un sous-domaine pour gitea.

![[Pasted image 20260505134317.png]]


## gitea

![[Pasted image 20260505134328.png]]


J'ai trouvé le mot de passe de cody.

![[Pasted image 20260505143207.png]]

Rien de trouvé sur le site Gitea.


## suid



Le mot de passe :


![[Pasted image 20260505143933.png]]

![[Pasted image 20260505144614.png]]


Si je fais ça :

`sudo python3 /opt/scripts/system-checkup.py docker-inspect '{{json .}}' gitea | jq .`



![[Pasted image 20260505145242.png]]


Je trouve un mdp.

![[Pasted image 20260505145305.png]]


Je suis capable de se connecter à administrator sur Gitea avec le mdp.

Dans le script system-checkup.py, on retrouve que ça appelle ce script :

![[Pasted image 20260505151923.png]]




Si on run full-checkup :

![[Pasted image 20260505152734.png]]



Ce qu’on peut faire, c’est créer un nouveau full-checkup.sh qui est un reverse shell. Lorsqu’on run le paramètre full-checkup.sh, ça run le script dans le folder actuel.

![[Pasted image 20260505154136.png]]

![[Pasted image 20260505154201.png]]

![[Pasted image 20260505154156.png]]
