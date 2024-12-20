---

layout: post  
title: Very Obfuscated Python Code  
subtitle: Write-Up pour le Nobrackets CTF  
tags: [writeup, reverse-engineering, memory-dump, CTF]  
comments: true

---

Salut les hackers ! Voici mon write-up sur le challenge **Very Obfuscated Python Code** du Nobrackets CTF. Ce défi nous emmène dans le monde merveilleux de l'obfuscation avec un petit échantillon d'obfuscation forte réalisée avec PyArmor. On adore, non ? 😏

## Scénario ✨

Le challenge fournit un conteneur Docker contenant une API Flask protégée par un mot de passe secret. Objectif : contourner l'obfuscation et récupérer le mot de passe pour décrocher le flag.

Première étape : lancement du conteneur Docker.

```bash
$ cd Dockerfile
$ docker build -t obfuscated-script-runner .
$ docker run -d --name my_obfuscated_container -v "$(pwd)/dist:/app" obfuscated-script-runner
$ docker logs my_obfuscated_container
```

On obtient les informations suivantes :

```bash
 * Running on http://127.0.0.1:8080
 * Running on http://172.17.0.2:8080
```

L’API exige un mot de passe pour les requêtes POST :

```bash
$ curl -X GET http://172.17.0.2:8080
{
  "message": "Il faut faire une requête POST avec un payload de type {'password': 'password \u00e0 remplacer'}"
}

$ curl -X POST http://172.17.0.2:8080 -H "Content-Type: application/json" -d '{"password": "remplacer"}'
{
  "message": "Login failed, Pas si facile, n'est t'il pas ? Le bon mot de passe s'il vous plait"
}
```

## Analyse et exploitation 🕵️

On décide de fouiller l’environnement du conteneur pour récupérer des informations runtime. Pour cela, on utilise un script Python de dump de mémoire.

### Étape 1 : Accès au conteneur

On copie le script dans le conteneur et on l’exécute :

```bash
$ docker cp memdump.py my_obfuscated_container:/app/
$ docker exec -it my_obfuscated_container /bin/sh
# python memdump.py 1
```

Le script extrait les régions mémoire lisibles et crée un fichier dump.

### Étape 2 : Analyse du dump

On analyse le fichier dump pour chercher des indices utiles :

```bash
$ strings -n 10 1.dump | grep -i VeryObfuscatedpythoncode -C 5
is_argument
_type_check
parameters
MatchOr(pattern* patterns)
VeryObfuscatedpythoncode
pyarmor_runtime.so
PyArmor v8+ runtime module
pyarmor_runtime_000000
<frozen python_secret>
```

Et bingo ! On tombe sur le mot de passe obfusqué : **VeryObfuscatedpythoncode**.

### Étape 3 : Soumission du mot de passe

On utilise ce mot de passe pour interroger l’API :

```bash
$ curl -X POST http://172.17.0.2:8080 -H "Content-Type: application/json" -d '{"password": "VeryObfuscatedpythoncode"}'
{
  "message": "NBCTF{13_mystere_reste_entier}"
}
```

Et voilà, flag récupéré : **NBCTF{13_mystere_reste_entier}** ! 🎉

## Conclusion 🏁

Ce challenge met en évidence l’importance de la sécurité des environnements runtime et de l’obfuscation. Bien que PyArmor soit puissant, il ne protège pas contre l’accès direct à la mémoire.

Un grand merci à l’équipe du Nobrackets CTF pour ce défi enrichissant. À la prochaine pour d’autres aventures hacking ! 😉
