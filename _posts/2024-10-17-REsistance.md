---

layout: post  
title: REsistance  
subtitle: Write-Up pour le Nobrackets CTF 
tags: [writeup, reverse-engineering, CTF, Endeavxor]  
comments: true

---

Salut à tous ! Aujourd'hui, je vais vous parler du challenge **REsistance** du NoBracket CTF, un challenge de reverse engineering qui nous met au défi de trouver un flag caché dans un fichier binaire. Voici les étapes détaillées que j'ai suivies pour résoudre ce défi.

## Analyse initiale

Le challenge commence avec un fichier binaire nommé `binary`. Le tag *reverse* nous indique qu'il s'agit d'un challenge de rétro-ingénierie. L'énoncé nous donne aussi un indice : "Les chaînes de caractères peuvent être intéressantes..."

C'est souvent le genre d'indice qui indique que le flag est probablement encodé ou caché quelque part dans le binaire. Cela m'a fait penser à deux outils utiles : **strings** pour extraire les chaînes de caractères lisibles d'un fichier, et **Dogbolt**, un décompilateur en ligne très pratique pour analyser rapidement le code binaire.

## Étape 1 : Utilisation de `strings`

Avant de faire du reverse engineering profond, j'ai lancé une première analyse rapide avec la commande `strings` pour voir s'il y avait des chaînes de caractères significatives directement visibles dans le fichier binaire. Cette commande permet d'extraire toutes les chaînes de caractères lisibles dans un fichier.

```bash
strings binary
```

Cependant, la commande `strings` ne m'a pas donné de résultat utile directement. Il n'y avait pas de flag visible dans les chaînes de caractères. C'est à ce moment que j'ai décidé d'utiliser **Dogbolt** pour aller plus loin dans l'analyse.

## Étape 2 : Analyse avec Dogbolt

Comme je ne pouvais rien extraire directement avec `strings`, j'ai uploadé le fichier sur **Dogbolt**, un outil en ligne qui permet de décompiler ou désassembler des fichiers binaires. Cela permet de voir une version plus lisible du code machine, ce qui aide à comprendre ce que fait le programme.

J'ai sélectionné **Binary Ninja** comme décompilateur, car il est performant pour l'analyse des binaires. Une fois le fichier uploadé et analysé, j'ai récupéré l'output suivant :

```c
void* const var_58 = "NBCTF{S7";
void* const var_50 = "R1NGs_GR";
void* const var_48 = "eP_for_7";
void* const var_40 = "he_W1N!}";
```

Dans cet extrait, on peut clairement voir quatre variables qui contiennent des parties du flag. Mais avant d'aller plus loin, j'ai voulu mieux comprendre le contexte dans lequel ces chaînes apparaissent dans le code.

## Étape 3 : Analyse du code principal

En explorant plus en détail l'output de **Dogbolt**, j'ai découvert la fonction `main()` du programme, qui contient la logique principale. Voici à quoi elle ressemble :

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

### Ce que fait le code :

1. **Affichage** : Le programme demande à l'utilisateur d'entrer le flag avec `printf("Entrez le flag : ");`.
2. **Lecture du flag** : Il utilise `fgets()` pour lire l'input de l'utilisateur.
3. **Vérification de la longueur** : Si la longueur du flag est différente de 32 caractères (`strlen(&buf) != 0x20`), il affiche "Flag incorrect".
4. **Comparaison des segments** : Ensuite, il compare chaque segment du flag entré par l'utilisateur avec les morceaux de flag contenus dans les variables `var_58`, `var_50`, `var_48`, et `var_40`.
5. **Validation** : Si toutes les parties correspondent, il affiche "Flag correct ! Bien jou".

## Étape 4 : Reconstruction du flag

Grâce à l'analyse du code, j'ai pu confirmer que les quatre variables contenaient bien des parties du flag. Voici comment elles se présentent :

- `"NBCTF{S7"`
- `"R1NGs_GR"`
- `"eP_for_7"`
- `"he_W1N!}"`

En les assemblant, on obtient le flag suivant :

```text
NBCTF{S7R1NGs_GReP_for_7he_W1N!}
```

C'est un flag bien formaté qui suit la structure habituelle des flags du CTF.

## Conclusion

Le challenge **REsistance** était un excellent exercice de reverse engineering. Il m'a permis d'utiliser des outils comme **Dogbolt** et d'analyser la structure d'un fichier binaire pour trouver un flag caché. Les chaînes de caractères étaient effectivement la clé pour résoudre ce défi, comme le suggérait l'indice dans l'énoncé. 

Merci à **Endeavxor** pour ce challenge ! J'ai beaucoup apprécié le processus de reverse engineering, et j'espère que ce write-up aidera d'autres participants à comprendre la méthodologie utilisée pour arriver au flag.

Flag : **NBCTF{S7R1NGs_GReP_for_7he_W1N!}**
