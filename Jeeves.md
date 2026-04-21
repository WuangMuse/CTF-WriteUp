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


# Recon


<img width="957" height="807" alt="image" src="https://github.com/user-attachments/assets/c11a3b28-f227-4e59-82df-a00d1c2ab95c" />





# Exploit



## port 135 rpc


J'ai besoin de credentials pour se connecter.

<img width="686" height="161" alt="image" src="https://github.com/user-attachments/assets/3255e488-ebcb-4d7c-899c-efca53e59fcf" />




## port 445 

Besoin de credentials pour accéder aux ressources.

<img width="651" height="173" alt="image" src="https://github.com/user-attachments/assets/31b396ec-68ee-48cb-8eab-342b0927dfbd" />



## port 80

Le site web au port 80.

<img width="962" height="553" alt="image" src="https://github.com/user-attachments/assets/d5ef9b4d-5a35-49a4-86fb-7f137c060cde" />



Lorsque j'essaie de rechercher une image de chat, j'ai une erreur.

<img width="1135" height="708" alt="image" src="https://github.com/user-attachments/assets/a904f081-e462-44bb-b668-606643585f14" />




### feroxbuster et dirbsearch

Je ne trouve rien d'intéressant avec ces deux outils.

<img width="828" height="672" alt="image" src="https://github.com/user-attachments/assets/733eab82-8b7e-4aa5-aab9-e3184759c097" />


<img width="889" height="269" alt="image" src="https://github.com/user-attachments/assets/641f1318-ee88-4112-b34e-9e855f4c52cb" />



## port 50000

Rien.

<img width="1100" height="556" alt="image" src="https://github.com/user-attachments/assets/3ca873c3-d112-443b-85b8-daba775ec40c" />



### feroxbuster et dfirbuster

Les deux outils ne retournent rien.

<img width="956" height="643" alt="image" src="https://github.com/user-attachments/assets/09b34bdb-ed32-4955-a5ef-f6038f896c9c" />




<img width="952" height="392" alt="image" src="https://github.com/user-attachments/assets/b04f7c31-71a0-4235-9e45-c651f8da2787" />




Si on utilise cette liste : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt, on obtient plusieurs résultats.

<img width="1006" height="799" alt="image" src="https://github.com/user-attachments/assets/1c7dae8e-b9d6-4a54-84df-51fdddfeb26e" />



## jenkins

Quand je visite askjeeves, je me retrouve dans jenkins.

<img width="1230" height="649" alt="image" src="https://github.com/user-attachments/assets/660911a5-f1b2-46e6-ad49-bf32e575a27f" />


Je retrouve la version de jenkins.

<img width="778" height="300" alt="image" src="https://github.com/user-attachments/assets/cccec75d-99d1-4361-bbaf-27f1b00ea6ae" />



On peut rouler du code.

<img width="1080" height="250" alt="image" src="https://github.com/user-attachments/assets/ec482a38-cca4-4afd-b199-85601fd43d95" />


Je prends un reverse shell en groovy, j'ouvre un listener et je l'exécute.

<img width="1046" height="482" alt="image" src="https://github.com/user-attachments/assets/ebdef6e2-1e1e-4cfe-91b8-c4f50f93410d" />



Par contre, je ne peux pas mettre de commandes dans mon shell.

<img width="717" height="313" alt="image" src="https://github.com/user-attachments/assets/dfab3280-3e46-4e6f-915a-b734d805203f" />


J'essaie avec un cmd plutôt qu'un powershell et ça marche.

<img width="741" height="335" alt="image" src="https://github.com/user-attachments/assets/67fce039-4b85-482b-b94d-9486f80b9f3e" />



Après on peut retrouver le flag de notre usager qu'on contrôle.

<img width="575" height="308" alt="image" src="https://github.com/user-attachments/assets/0e20927d-aa1a-4aff-b635-212529bc02d6" />





# PrivEsc



## kdbx


Je retrouve un kdbx dans le répertoire documents.

<img width="617" height="549" alt="image" src="https://github.com/user-attachments/assets/90a2cffe-d1ec-4fe1-9e68-8c04089ac43c" />


### transferere fichier de windows a linux

J'ouvre un serveur smb.

<img width="777" height="121" alt="image" src="https://github.com/user-attachments/assets/05dc7115-f0b5-4c02-9c4d-4ee54a1ec9eb" />


Sur windows, je transfère mon fichier comme suit.

<img width="800" height="314" alt="image" src="https://github.com/user-attachments/assets/3e113465-0599-4ab6-9857-509823240d76" />



### crack le .kdbx

J'extrais le hash du fichier kdbx.

<img width="706" height="268" alt="image" src="https://github.com/user-attachments/assets/9b9cdfc0-6e72-481e-9b6e-ccfac0ef2322" />


On crack le hash ensuite avec john.

<img width="848" height="364" alt="image" src="https://github.com/user-attachments/assets/8d9adc6b-3f94-4bfe-ae0d-bd52e38ed610" />




J'ouvre le kdbx et j'insère le mot de passe trouvé.

<img width="1065" height="581" alt="image" src="https://github.com/user-attachments/assets/a364b0de-3c9a-4cc9-9adc-0666b8fd3ef7" />


<img width="860" height="359" alt="image" src="https://github.com/user-attachments/assets/40886ccb-9a9a-46da-99d8-4a49689d7278" />

Je sauvegarde les mots de passe dans un fichier.

<img width="489" height="415" alt="image" src="https://github.com/user-attachments/assets/263b96a9-7be0-4129-8032-71a30bd86cd4" />


Le password spray n'a pas révélé d'accès.


En retournant sur le kdbx, je trouve un hash ntlm.

<img width="816" height="547" alt="image" src="https://github.com/user-attachments/assets/c8c7c21c-bf36-4037-9b96-b82d1d6f860f" />



Le hash nt fonctionne pour l'usager administrator.

<img width="870" height="254" alt="image" src="https://github.com/user-attachments/assets/ffb71b96-5baa-49ac-9537-c47f116d2f35" />


## ps exec

Il n'y a pas de ssh ni de winrm. Toutefois, on a du smb qui est capable d'être connecté. J'utilise donc psexec.

`impacket-psexec -hashes :e0fb1fb85756c24235ff238cbe81fe00 administrator@10.129.23.232`

<img width="835" height="338" alt="image" src="https://github.com/user-attachments/assets/a1710b3c-f530-4b31-8314-5db2399aa23a" />


Pour avoir le flag, il faut regarder l'alternate data stream. L'alternate data stream est des données qui sont attachées à un fichier et ce n'est pas visible par dir (sans /R) ni par l'explorateur de fichiers.

<img width="789" height="598" alt="image" src="https://github.com/user-attachments/assets/d99b3d3f-8555-4030-894a-a189497153f9" />




