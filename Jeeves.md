---
date: 2026-04-21
tags:
  - htb
  - medium
  - windows
  - jenkins
  - jeeves
  - kdbx
---
- [[#Recon|Recon]]
- [[#Exploit|Exploit]]
	- [[#Exploit#port 135 rpc|port 135 rpc]]
	- [[#Exploit#port 445|port 445]]
	- [[#Exploit#port 80|port 80]]
		- [[#port 80#feroxbuster et dirbsearch|feroxbuster et dirbsearch]]
	- [[#Exploit#port 50000|port 50000]]
		- [[#port 50000#feroxbuster et dfirbuster|feroxbuster et dfirbuster]]
	- [[#Exploit#jenkins|jenkins]]
- [[#PrivEsc|PrivEsc]]
	- [[#PrivEsc#kdbx|kdbx]]
		- [[#kdbx#transferere fichier de windows a linux|transferere fichier de windows a linux]]
		- [[#kdbx#crack le .kdbx|crack le .kdbx]]
	- [[#PrivEsc#ps exec|ps exec]]

# Recon


![[Pasted image 20260421100056.png]]




# Exploit



## port 135 rpc


J'ai besoin de credentials pour se connecter.

![[Pasted image 20260421100212.png]]



## port 445 

Besoin de credentials pour accéder aux ressources.

![[Pasted image 20260421100657.png]]



## port 80

Le site web au port 80.

![[Pasted image 20260421101154.png]]


Lorsque j'essaie de rechercher une image de chat, j'ai une erreur.

![[Pasted image 20260421101236.png]]



### feroxbuster et dirbsearch

Je ne trouve rien d'intéressant avec ces deux outils.

![[Pasted image 20260421101726.png]]


![[Pasted image 20260421101733.png]]



## port 50000

Rien.

![[Pasted image 20260421104036.png]]


### feroxbuster et dfirbuster

Les deux outils ne retournent rien.

![[Pasted image 20260421105256.png]]



![[Pasted image 20260421110238.png]]



Si on utilise cette liste : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt, on obtient plusieurs résultats.

![[Pasted image 20260421112645.png]]


## jenkins

Quand je visite askjeeves, je me retrouve dans jenkins.

![[Pasted image 20260421112746.png]]


Je retrouve la version de jenkins.

![[Pasted image 20260421113149.png]]


On peut rouler du code.

![[Pasted image 20260421114212.png]]

Je prends un reverse shell en groovy, j'ouvre un listener et je l'exécute.

![[Pasted image 20260421114234.png]]


Par contre, je ne peux pas mettre de commandes dans mon shell.

![[Pasted image 20260421115705.png]]


J'essaie avec un cmd plutôt qu'un powershell et ça marche.

![[Pasted image 20260421115759.png]]


Après on peut retrouver le flag de notre usager qu'on contrôle.

![[Pasted image 20260421120606.png]]





# PrivEsc



## kdbx


Je retrouve un kdbx dans le répertoire documents.

![[Pasted image 20260421120653.png]]


### transferere fichier de windows a linux

J'ouvre un serveur smb.

![[Pasted image 20260421121233.png]]

Sur windows, je transfère mon fichier comme suit.

![[Pasted image 20260421121307.png]]


### crack le .kdbx

J'extrais le hash du fichier kdbx.

![[Pasted image 20260421121415.png]]


On crack le hash ensuite avec john.

![[Pasted image 20260421122606.png]]



J'ouvre le kdbx et j'insère le mot de passe trouvé.

![[Pasted image 20260421123105.png]]

![[Pasted image 20260421123243.png]]
Je sauvegarde les mots de passe dans un fichier.

![[Pasted image 20260421123350.png]]

Le password spray n'a pas révélé d'accès.


En retournant sur le kdbx, je trouve un hash ntlm.

![[Pasted image 20260421124848.png]]


Le hash nt fonctionne pour l'usager administrator.

![[Pasted image 20260421125152.png]]


## ps exec

Il n'y a pas de ssh ni de winrm. Toutefois, on a du smb qui est capable d'être connecté. J'utilise donc psexec.

`impacket-psexec -hashes :e0fb1fb85756c24235ff238cbe81fe00 administrator@10.129.23.232`

![[Pasted image 20260421125540.png]]


Pour avoir le flag, il faut regarder l'alternate data stream. L'alternate data stream est des données qui sont attachées à un fichier et ce n'est pas visible par dir (sans /R) ni par l'explorateur de fichiers.

![[Pasted image 20260421131513.png]]



