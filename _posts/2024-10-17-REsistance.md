---

layout: post  
title: REsistance  
subtitle: Write-Up pour le Nobrackets CTF  
tags: [writeup, reverse-engineering, CTF, Endeavxor]  
comments: true

---

Salut à tous ! Aujourd'hui, je vais vous parler du challenge **REsistance** du NoBracket CTF, un challenge de reverse engineering qui nous met au défi de trouver un flag caché dans un fichier binaire. Prêt à plonger dans les méandres du code ? Voici les étapes que j'ai suivies pour résoudre ce défi. 🚀

## Analyse initiale 🔍

Le challenge commence avec un fichier binaire nommé `binary`. Le tag *reverse* nous donne un petit indice sur la nature du défi. Mais ce qui attire mon attention, c'est la mention des "chaînes de caractères intéressantes"... Hmm, ça sent le flag bien caché dans le binaire ! 🕵️‍♂️

Quand je vois ce genre d'indice, je pense immédiatement à deux outils : **strings** pour jeter un coup d'œil aux chaînes lisibles, et **Dogbolt**, mon allié pour la rétro-ingénierie rapide.

## Étape 1 : Utilisation de `strings` 🧵

Avant de creuser profondément, je commence par une analyse basique avec `strings`. C'est un bon moyen de voir s'il y a des indices évidents qui traînent dans le binaire.

```bash
strings binary
```

Malheureusement, rien d'intéressant n'est ressorti. Aucune trace de flag visible à l'œil nu... Pas de panique, on a encore quelques tours dans notre sac. Direction **Dogbolt** pour des choses plus sérieuses ! 💻

## Étape 2 : Analyse avec Dogbolt 🧠

N'ayant rien trouvé avec `strings`, j'upload le binaire sur **Dogbolt** pour le décompiler. J'ai utilisé **Binary Ninja** comme décompilateur, car il est particulièrement efficace pour analyser les binaires. Une fois le fichier analysé, je tombe sur cet extrait juteux :

```c
void* const var_58 = "NBCTF{S7";
void* const var_50 = "R1NGs_GR";
void* const var_48 = "eP_for_7";
void* const var_40 = "he_W1N!}";
```

Bingo ! 🎯 On peut clairement voir des morceaux de flag ici, mais je préfère m'assurer du contexte avant de célébrer trop vite. Continuons à décortiquer le code pour comprendre comment il est utilisé.

## Étape 3 : Analyse du code principal 🔑

En explorant le code de **Dogbolt**, je tombe sur la fonction `main()`. C'est là que tout se passe. Voici ce que j'ai trouvé :

```c
int32_t main(int32_t argc, char** argv, char** envp)
{
    void* const var_58 = "NBCTF{S7";
    void* const var_50 = "R1NGs_GR";
    void* const var_48 = "eP_for_7";
    void* const var_40 = "he_W1N!}";
    int32_t var_68 = 3;
    int32_t var_64 = 1;
    int32_t var_60 = 2;
    int32_t var_5c = 0;
    printf("Entrez le flag : ");
    void buf;
    
    if (fgets(&buf, 0x64, stdin) == 0)
    {
        fwrite("Erreur de lecture.\n", 1, 0x13, stderr);
        return 1;
    }
    
    *(&buf + strcspn(&buf, &data_204e)) = 0;
    
    if (strlen(&buf) != 0x20)
    {
        puts("Flag incorrect.");
        return 1;
    }
    
    int32_t var_c_1 = 0;
    
    while (true)
    {
        if (var_c_1 > 3)
        {
            puts("Flag correct ! Bien jou");
            return 0;
        }
        
        int32_t rax_10 = &var_68[var_c_1];
        
        if (strncmp((&buf + (rax_10 << 3)), &var_58[rax_10], 8) != 0)
            break;
        
        var_c_1 += 1;
    }
    
    puts("Flag incorrect.");
    return 1;
}
```

### Ce que fait le code 🔍 :

1. **Demande de flag** : Le programme demande à l'utilisateur de saisir un flag via `printf("Entrez le flag : ");`.
2. **Lecture de l'entrée** : Il utilise `fgets()` pour lire l'input de l'utilisateur.
3. **Vérification de la longueur** : Si la longueur du flag n'est pas de 32 caractères, il affiche "Flag incorrect".
4. **Comparaison des segments** : Ensuite, il compare chaque segment du flag entré par l'utilisateur avec les morceaux de flag dans `var_58`, `var_50`, `var_48`, et `var_40`.
5. **Validation finale** : Si tout correspond, le programme valide avec "Flag correct ! Bien jou". 🎉

## Étape 4 : Reconstruction du flag 🧩

Les morceaux de flag dans le code sont assez clairs. Voici comment ils se présentent :

- `"NBCTF{S7"`
- `"R1NGs_GR"`
- `"eP_for_7"`
- `"he_W1N!}"`

En les assemblant, j'obtiens le flag complet :

```text
NBCTF{S7R1NGs_GReP_for_7he_W1N!}
```

C'est toujours satisfaisant quand tout se recolle parfaitement. 🏆

## Conclusion 🎯

Le challenge **REsistance** était un excellent exercice de reverse engineering. J'ai utilisé **Dogbolt** pour analyser le binaire et extraire les chaînes de caractères cachées, puis j'ai suivi la logique du programme pour comprendre comment le flag était validé. Au final, tout se résume à de la bonne vieille analyse de code et à quelques indices bien cachés.

Merci à **Endeavxor** pour ce challenge vraiment captivant. J'espère que ce write-up aidera d'autres participants à dénouer les mystères du reverse engineering.

Flag : **NBCTF{S7R1NGs_GReP_for_7he_W1N!}**
