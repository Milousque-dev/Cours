# Chapitre 4 : Structures de contrôle

## Table des matières

1. [Introduction aux structures de contrôle](#introduction-aux-structures-de-contrôle)
2. [Instructions conditionnelles : if, else, else if](#instructions-conditionnelles--if-else-else-if)
3. [L'instruction switch](#linstruction-switch)
4. [Boucle while](#boucle-while)
5. [Boucle do-while](#boucle-do-while)
6. [Boucle for](#boucle-for)
7. [Instructions de saut : break, continue, goto](#instructions-de-saut--break-continue-goto)
8. [Boucles imbriquées](#boucles-imbriquées)
9. [Patterns courants et idiomes](#patterns-courants-et-idiomes)
10. [Comparaison avec Python/JS/C#/Go](#comparaison-avec-pythonjscgo)
11. [Exercices du chapitre](#exercices-du-chapitre)

---

## Introduction aux structures de contrôle

### Flux d'exécution

Par défaut, un programme C s'exécute **séquentiellement**, instruction par instruction. Les structures de contrôle permettent de modifier ce flux :

```
Structures de contrôle
├── Séquence (défaut)
│   instruction1;
│   instruction2;
│   instruction3;
│
├── Sélection (choix)
│   ├── if / else / else if
│   └── switch / case
│
├── Itération (répétition)
│   ├── while
│   ├── do-while
│   └── for
│
└── Saut (rupture de flux)
    ├── break
    ├── continue
    ├── goto
    └── return
```

### Blocs d'instructions

Un **bloc** est un groupe d'instructions entre accolades `{ }` :

```c
{
    // Ceci est un bloc
    int x = 10;        // Variable locale au bloc
    printf("%d\n", x);
}
// x n'existe plus ici

// Une seule instruction peut omettre les accolades
if (condition)
    printf("Une seule ligne\n");

// Mais c'est souvent source d'erreurs :
if (condition)
    printf("Ligne 1\n");
    printf("Ligne 2\n");  // S'exécute TOUJOURS (pas dans le if!)

// Recommandation : TOUJOURS utiliser les accolades
if (condition) {
    printf("Ligne 1\n");
    printf("Ligne 2\n");
}
```

---

## Instructions conditionnelles : if, else, else if

### Syntaxe de base

```c
// if simple
if (condition) {
    // Code exécuté si condition est vraie (non nulle)
}

// if-else
if (condition) {
    // Code si vrai
} else {
    // Code si faux
}

// if-else if-else (chaîne de conditions)
if (condition1) {
    // Code si condition1 vraie
} else if (condition2) {
    // Code si condition1 fausse ET condition2 vraie
} else if (condition3) {
    // Code si conditions 1 et 2 fausses ET condition3 vraie
} else {
    // Code si toutes les conditions sont fausses
}
```

### Exemples pratiques

```c
#include <stdio.h>

int main(void) {
    int age = 25;
    
    // Classification d'âge
    if (age < 0) {
        printf("Âge invalide\n");
    } else if (age < 13) {
        printf("Enfant\n");
    } else if (age < 18) {
        printf("Adolescent\n");
    } else if (age < 65) {
        printf("Adulte\n");
    } else {
        printf("Senior\n");
    }
    
    // Conditions multiples avec opérateurs logiques
    int x = 15;
    if (x >= 10 && x <= 20) {
        printf("%d est entre 10 et 20\n", x);
    }
    
    if (x < 0 || x > 100) {
        printf("Hors bornes\n");
    } else {
        printf("Dans les bornes\n");
    }
    
    // Conditions imbriquées
    int note = 75;
    char lettre;
    
    if (note >= 60) {
        if (note >= 90) {
            lettre = 'A';
        } else if (note >= 80) {
            lettre = 'B';
        } else if (note >= 70) {
            lettre = 'C';
        } else {
            lettre = 'D';
        }
    } else {
        lettre = 'F';
    }
    printf("Note : %c\n", lettre);
    
    return 0;
}
```

### Pièges et bonnes pratiques

```c
#include <stdio.h>

int main(void) {
    int x = 0;
    
    // PIÈGE 1 : Affectation au lieu de comparaison
    if (x = 5) {  // AFFECTATION! x devient 5, condition vraie
        printf("Toujours exécuté!\n");
    }
    
    // Solution : constante à gauche (Yoda condition)
    if (5 == x) {  // "5 = x" causerait une erreur de compilation
        printf("x vaut 5\n");
    }
    
    // PIÈGE 2 : Dangling else
    int a = 1, b = 0;
    if (a)
        if (b)
            printf("a et b\n");
    else  // À quel if appartient ce else ? Au plus proche (if(b))!
        printf("Ce else appartient à if(b)\n");
    
    // Solution : toujours utiliser des accolades
    if (a) {
        if (b) {
            printf("a et b\n");
        }
    } else {
        printf("Maintenant c'est clair : pas a\n");
    }
    
    // PIÈGE 3 : Point-virgule après if
    if (x > 0);  // Instruction vide! Le bloc suivant s'exécute toujours
    {
        printf("Pas conditionnel!\n");
    }
    
    // PIÈGE 4 : Comparaison de chaînes
    char str[] = "hello";
    // if (str == "hello")  // Compare les ADRESSES, pas le contenu!
    // Solution :
    if (strcmp(str, "hello") == 0) {
        printf("Chaînes égales\n");
    }
    
    return 0;
}
```

### Conditions avec valeurs booléennes implicites

```c
#include <stdio.h>
#include <stdbool.h>

int main(void) {
    int n = 42;
    int *ptr = &n;
    char str[] = "Hello";
    
    // Tester si non-nul (idiome C)
    if (n) {  // Équivalent à : if (n != 0)
        printf("n est non-nul\n");
    }
    
    // Tester si nul
    if (!n) {  // Équivalent à : if (n == 0)
        printf("n est nul\n");
    }
    
    // Tester un pointeur
    if (ptr) {  // Équivalent à : if (ptr != NULL)
        printf("Pointeur valide\n");
    }
    
    if (!ptr) {  // Équivalent à : if (ptr == NULL)
        printf("Pointeur nul\n");
    }
    
    // Tester une chaîne non vide
    if (*str) {  // Premier caractère non nul
        printf("Chaîne non vide\n");
    }
    
    if (str[0] == '\0') {  // Plus explicite
        printf("Chaîne vide\n");
    }
    
    return 0;
}
```

---

## L'instruction switch

### Syntaxe et fonctionnement

Le `switch` permet de tester une variable contre plusieurs valeurs constantes :

```c
switch (expression) {
    case constante1:
        // Code pour constante1
        break;
    case constante2:
        // Code pour constante2
        break;
    case constante3:
    case constante4:
        // Code pour constante3 OU constante4 (fall-through intentionnel)
        break;
    default:
        // Code si aucune correspondance
        break;
}
```

### Exemple complet

```c
#include <stdio.h>

int main(void) {
    int jour = 3;
    
    switch (jour) {
        case 1:
            printf("Lundi\n");
            break;
        case 2:
            printf("Mardi\n");
            break;
        case 3:
            printf("Mercredi\n");
            break;
        case 4:
            printf("Jeudi\n");
            break;
        case 5:
            printf("Vendredi\n");
            break;
        case 6:
        case 7:
            printf("Week-end!\n");  // 6 et 7 partagent le même code
            break;
        default:
            printf("Jour invalide\n");
            break;
    }
    
    // Switch avec caractères
    char grade = 'B';
    
    switch (grade) {
        case 'A':
            printf("Excellent!\n");
            break;
        case 'B':
            printf("Très bien\n");
            break;
        case 'C':
            printf("Bien\n");
            break;
        case 'D':
            printf("Passable\n");
            break;
        case 'F':
            printf("Échec\n");
            break;
        default:
            printf("Note invalide\n");
    }
    
    return 0;
}
```

### Le piège du fall-through

```c
#include <stdio.h>

int main(void) {
    int x = 2;
    
    // SANS break : fall-through (exécution en cascade)
    printf("Sans break :\n");
    switch (x) {
        case 1:
            printf("Un\n");
        case 2:
            printf("Deux\n");
        case 3:
            printf("Trois\n");
        default:
            printf("Autre\n");
    }
    // Affiche : Deux, Trois, Autre
    
    // AVEC break : comportement attendu
    printf("\nAvec break :\n");
    switch (x) {
        case 1:
            printf("Un\n");
            break;
        case 2:
            printf("Deux\n");
            break;
        case 3:
            printf("Trois\n");
            break;
        default:
            printf("Autre\n");
            break;
    }
    // Affiche : Deux
    
    // Fall-through INTENTIONNEL (documenter avec commentaire)
    char c = 'a';
    switch (c) {
        case 'a':
        case 'e':
        case 'i':
        case 'o':
        case 'u':
        case 'A':
        case 'E':
        case 'I':
        case 'O':
        case 'U':
            printf("Voyelle\n");
            break;
        default:
            printf("Consonne ou autre\n");
    }
    
    return 0;
}
```

### Limitations du switch

```c
// Le switch ne supporte QUE :
// - Types entiers (int, char, enum)
// - Constantes (pas de variables ni d'expressions)
// - Égalité (pas de <, >, plages)

// INVALIDE :
switch (x) {
    case y:         // ERREUR : y n'est pas une constante
    case 1..10:     // ERREUR : pas de plages (sauf extension GCC)
    case x > 0:     // ERREUR : pas d'expressions
}

// Pour les flottants : utiliser if-else
double d = 3.14;
// switch (d) { }  // ERREUR : pas de switch sur flottants

// Pour les chaînes : utiliser if-else avec strcmp
char *str = "hello";
// switch (str) { }  // Compare les pointeurs, pas le contenu!

// Alternative pour les chaînes
if (strcmp(str, "hello") == 0) {
    // ...
} else if (strcmp(str, "world") == 0) {
    // ...
}
```

### Variables dans les cases (C99+)

```c
switch (x) {
    case 1: {
        // Les accolades créent une portée pour la variable
        int local = 10;
        printf("%d\n", local);
        break;
    }
    case 2: {
        int local = 20;  // OK : portée différente
        printf("%d\n", local);
        break;
    }
}
```

---

## Boucle while

### Syntaxe

```c
while (condition) {
    // Corps de la boucle
    // Exécuté tant que condition est vraie (non nulle)
}
```

### Fonctionnement

1. Évaluer la condition
2. Si vraie (non nulle) → exécuter le corps → retour à l'étape 1
3. Si fausse (nulle) → sortir de la boucle

```c
#include <stdio.h>

int main(void) {
    // Compteur simple
    int i = 0;
    while (i < 5) {
        printf("%d ", i);
        i++;
    }
    printf("\n");  // 0 1 2 3 4
    
    // Compte à rebours
    int n = 5;
    while (n > 0) {
        printf("%d ", n);
        n--;
    }
    printf("Décollage!\n");  // 5 4 3 2 1 Décollage!
    
    // Somme de 1 à n
    int somme = 0;
    int num = 1;
    while (num <= 100) {
        somme += num;
        num++;
    }
    printf("Somme 1-100 : %d\n", somme);  // 5050
    
    // Lecture jusqu'à condition
    int valeur;
    printf("Entrez des nombres (0 pour arrêter) :\n");
    while (1) {  // Boucle infinie
        scanf("%d", &valeur);
        if (valeur == 0) {
            break;  // Sort de la boucle
        }
        printf("Vous avez entré : %d\n", valeur);
    }
    
    return 0;
}
```

### Idiomes courants

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    // Parcourir une chaîne
    char str[] = "Hello";
    int i = 0;
    while (str[i] != '\0') {
        printf("%c ", str[i]);
        i++;
    }
    printf("\n");
    
    // Avec pointeur (idiome C classique)
    char *p = str;
    while (*p) {  // *p != '\0' implicite
        printf("%c ", *p);
        p++;
    }
    printf("\n");
    
    // Encore plus court
    p = str;
    while (*p) {
        printf("%c ", *p++);  // Affiche puis avance
    }
    printf("\n");
    
    // Lecture de fichier ligne par ligne
    FILE *f = fopen("test.txt", "r");
    if (f != NULL) {
        char buffer[256];
        while (fgets(buffer, sizeof(buffer), f) != NULL) {
            printf("%s", buffer);
        }
        fclose(f);
    }
    
    // Algorithme d'Euclide (PGCD)
    int a = 48, b = 18;
    while (b != 0) {
        int temp = b;
        b = a % b;
        a = temp;
    }
    printf("PGCD : %d\n", a);  // 6
    
    return 0;
}
```

---

## Boucle do-while

### Syntaxe

```c
do {
    // Corps de la boucle
    // Exécuté AU MOINS une fois
} while (condition);  // Note le point-virgule!
```

### Différence avec while

- **while** : teste AVANT d'exécuter (peut ne jamais exécuter)
- **do-while** : exécute PUIS teste (exécute au moins une fois)

```c
#include <stdio.h>

int main(void) {
    int x = 0;
    
    // while : condition fausse, corps jamais exécuté
    while (x > 0) {
        printf("while: %d\n", x);
        x--;
    }
    
    // do-while : corps exécuté une fois même si condition fausse
    do {
        printf("do-while: %d\n", x);
        x--;
    } while (x > 0);
    // Affiche : do-while: 0
    
    return 0;
}
```

### Cas d'usage typiques

```c
#include <stdio.h>

int main(void) {
    // 1. Validation d'entrée utilisateur
    int choix;
    do {
        printf("Menu:\n");
        printf("1. Option A\n");
        printf("2. Option B\n");
        printf("3. Quitter\n");
        printf("Votre choix : ");
        scanf("%d", &choix);
        
        if (choix < 1 || choix > 3) {
            printf("Choix invalide!\n\n");
        }
    } while (choix < 1 || choix > 3);
    
    printf("Vous avez choisi %d\n", choix);
    
    // 2. Demander confirmation
    char reponse;
    do {
        printf("Êtes-vous sûr ? (o/n) : ");
        scanf(" %c", &reponse);  // Espace pour ignorer whitespace
    } while (reponse != 'o' && reponse != 'n' && 
             reponse != 'O' && reponse != 'N');
    
    // 3. Nombre de chiffres d'un entier
    int n = 12345;
    int original = n;
    int chiffres = 0;
    do {
        chiffres++;
        n /= 10;
    } while (n > 0);
    printf("%d a %d chiffres\n", original, chiffres);
    
    return 0;
}
```

---

## Boucle for

### Syntaxe

```c
for (initialisation; condition; mise_à_jour) {
    // Corps de la boucle
}

// Équivalent while :
initialisation;
while (condition) {
    // Corps
    mise_à_jour;
}
```

### Exemples de base

```c
#include <stdio.h>

int main(void) {
    // Compteur classique
    for (int i = 0; i < 5; i++) {
        printf("%d ", i);
    }
    printf("\n");  // 0 1 2 3 4
    
    // Compte à rebours
    for (int i = 10; i > 0; i--) {
        printf("%d ", i);
    }
    printf("Décollage!\n");
    
    // Pas de 2
    for (int i = 0; i < 10; i += 2) {
        printf("%d ", i);
    }
    printf("\n");  // 0 2 4 6 8
    
    // Parcours de tableau
    int arr[] = {10, 20, 30, 40, 50};
    int taille = sizeof(arr) / sizeof(arr[0]);
    
    for (int i = 0; i < taille; i++) {
        printf("arr[%d] = %d\n", i, arr[i]);
    }
    
    // Parcours inverse
    for (int i = taille - 1; i >= 0; i--) {
        printf("arr[%d] = %d\n", i, arr[i]);
    }
    
    return 0;
}
```

### Formes avancées

```c
#include <stdio.h>

int main(void) {
    // Variables multiples
    for (int i = 0, j = 10; i < j; i++, j--) {
        printf("i=%d, j=%d\n", i, j);
    }
    
    // Parties optionnelles
    int k = 0;
    for (; k < 5; k++) {  // Initialisation vide
        printf("%d ", k);
    }
    printf("\n");
    
    for (int i = 0; i < 5; ) {  // Mise à jour vide
        printf("%d ", i);
        i++;
    }
    printf("\n");
    
    // Boucle infinie
    // for (;;) {  // Équivalent à while(1)
    //     // ...
    //     if (condition) break;
    // }
    
    // Parcours de chaîne avec for
    char str[] = "Hello";
    for (int i = 0; str[i] != '\0'; i++) {
        printf("%c ", str[i]);
    }
    printf("\n");
    
    // Avec pointeur
    for (char *p = str; *p; p++) {
        printf("%c ", *p);
    }
    printf("\n");
    
    return 0;
}
```

### Portée des variables de boucle

```c
#include <stdio.h>

int main(void) {
    // C99+ : variable déclarée dans le for
    for (int i = 0; i < 5; i++) {
        printf("%d ", i);
    }
    // printf("%d", i);  // ERREUR : i n'existe plus ici
    
    // C89 : déclarer avant
    int j;
    for (j = 0; j < 5; j++) {
        printf("%d ", j);
    }
    printf("\nAprès boucle : j = %d\n", j);  // j = 5 (accessible)
    
    // Réutilisation de la même variable
    for (int i = 0; i < 3; i++) {
        printf("Boucle 1 : %d\n", i);
    }
    for (int i = 10; i < 13; i++) {
        printf("Boucle 2 : %d\n", i);  // Nouveau i, pas de conflit
    }
    
    return 0;
}
```

---

## Instructions de saut : break, continue, goto

### break : sortie immédiate

`break` termine immédiatement la boucle (ou le switch) la plus proche :

```c
#include <stdio.h>

int main(void) {
    // Sortie anticipée
    for (int i = 0; i < 100; i++) {
        if (i == 5) {
            break;  // Sort de la boucle
        }
        printf("%d ", i);
    }
    printf("\n");  // 0 1 2 3 4
    
    // Recherche dans un tableau
    int arr[] = {5, 10, 15, 20, 25};
    int cible = 15;
    int trouve = 0;
    int index = -1;
    
    for (int i = 0; i < 5; i++) {
        if (arr[i] == cible) {
            trouve = 1;
            index = i;
            break;  // Inutile de continuer
        }
    }
    
    if (trouve) {
        printf("Trouvé à l'index %d\n", index);
    }
    
    // break ne sort que de la boucle la plus interne
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            if (j == 1) {
                break;  // Sort de la boucle j seulement
            }
            printf("(%d, %d) ", i, j);
        }
    }
    printf("\n");  // (0, 0) (1, 0) (2, 0)
    
    return 0;
}
```

### continue : passer à l'itération suivante

`continue` saute le reste du corps et passe à l'itération suivante :

```c
#include <stdio.h>

int main(void) {
    // Sauter certaines valeurs
    for (int i = 0; i < 10; i++) {
        if (i % 2 == 0) {
            continue;  // Saute les pairs
        }
        printf("%d ", i);
    }
    printf("\n");  // 1 3 5 7 9
    
    // Ignorer les lignes vides
    char *lignes[] = {"Hello", "", "World", "", "!"};
    for (int i = 0; i < 5; i++) {
        if (lignes[i][0] == '\0') {
            continue;  // Saute les chaînes vides
        }
        printf("%s\n", lignes[i]);
    }
    
    // Traitement avec filtrage
    for (int i = 1; i <= 20; i++) {
        if (i % 3 != 0) {
            continue;  // Ne traite que les multiples de 3
        }
        printf("%d est divisible par 3\n", i);
    }
    
    return 0;
}
```

### Différence break vs continue

```c
printf("break :\n");
for (int i = 0; i < 5; i++) {
    if (i == 3) break;
    printf("%d ", i);
}
printf("\n");  // 0 1 2

printf("continue :\n");
for (int i = 0; i < 5; i++) {
    if (i == 3) continue;
    printf("%d ", i);
}
printf("\n");  // 0 1 2 4
```

### goto : saut inconditionnel

`goto` permet de sauter à une étiquette (label). **Généralement déconseillé**, mais utile dans certains cas :

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    // CAS D'USAGE LÉGITIME 1 : Sortir de boucles imbriquées
    for (int i = 0; i < 10; i++) {
        for (int j = 0; j < 10; j++) {
            for (int k = 0; k < 10; k++) {
                if (/* condition */) {
                    goto sortie;  // Sort des 3 boucles d'un coup
                }
            }
        }
    }
    sortie:
    printf("Sorti des boucles\n");
    
    // CAS D'USAGE LÉGITIME 2 : Gestion d'erreurs avec nettoyage
    FILE *f1 = NULL;
    FILE *f2 = NULL;
    char *buffer = NULL;
    
    f1 = fopen("file1.txt", "r");
    if (f1 == NULL) {
        goto cleanup;  // Erreur : nettoyer et sortir
    }
    
    f2 = fopen("file2.txt", "w");
    if (f2 == NULL) {
        goto cleanup;
    }
    
    buffer = malloc(1024);
    if (buffer == NULL) {
        goto cleanup;
    }
    
    // Traitement normal...
    printf("Tout s'est bien passé\n");
    
    cleanup:
    // Point de nettoyage unique
    if (buffer) free(buffer);
    if (f2) fclose(f2);
    if (f1) fclose(f1);
    
    return 0;
}
```

### Quand utiliser goto ?

| Usage                            | Recommandé ?                                         |
|----------------------------------|------------------------------------------------------|
| Sortie de boucles multiples      | ✅ Acceptable                                        |
| Gestion d'erreurs avec nettoyage | ✅ Pattern courant en C                              |
| Saut vers l'avant                | ⚠️ À éviter généralement                             |
| Saut vers l'arrière              | ❌ Éviter (crée des boucles difficiles à comprendre) |
| Remplacer des boucles            | ❌ Jamais                                            |

---

## Boucles imbriquées

### Syntaxe et exemples

```c
#include <stdio.h>

int main(void) {
    // Table de multiplication
    printf("Table de multiplication 1-10:\n");
    for (int i = 1; i <= 10; i++) {
        for (int j = 1; j <= 10; j++) {
            printf("%4d", i * j);
        }
        printf("\n");
    }
    
    // Triangle d'étoiles
    int hauteur = 5;
    for (int i = 1; i <= hauteur; i++) {
        for (int j = 1; j <= i; j++) {
            printf("*");
        }
        printf("\n");
    }
    /*
    *
    **
    ***
    ****
    *****
    */
    
    // Matrice 2D
    int matrice[3][4] = {
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12}
    };
    
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 4; j++) {
            printf("%3d ", matrice[i][j]);
        }
        printf("\n");
    }
    
    return 0;
}
```

### Sortie de boucles imbriquées

```c
#include <stdio.h>
#include <stdbool.h>

int main(void) {
    // Méthode 1 : Variable drapeau
    bool trouve = false;
    for (int i = 0; i < 10 && !trouve; i++) {
        for (int j = 0; j < 10 && !trouve; j++) {
            if (/* condition */) {
                printf("Trouvé à (%d, %d)\n", i, j);
                trouve = true;
            }
        }
    }
    
    // Méthode 2 : goto (plus direct)
    for (int i = 0; i < 10; i++) {
        for (int j = 0; j < 10; j++) {
            if (/* condition */) {
                printf("Trouvé à (%d, %d)\n", i, j);
                goto fin_recherche;
            }
        }
    }
    fin_recherche:
    
    // Méthode 3 : Fonction séparée avec return
    // Voir section suivante
    
    return 0;
}
```

### Complexité des boucles imbriquées

```c
// O(n) - Linéaire
for (int i = 0; i < n; i++) {
    // ...
}

// O(n²) - Quadratique
for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) {
        // ...
    }
}

// O(n³) - Cubique
for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) {
        for (int k = 0; k < n; k++) {
            // ...
        }
    }
}

// O(n * m) - Selon les dimensions
for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
        // ...
    }
}
```

---

## Patterns courants et idiomes

### Boucle de lecture jusqu'à EOF

```c
#include <stdio.h>

int main(void) {
    int c;
    // getchar() retourne int, pas char (pour détecter EOF)
    while ((c = getchar()) != EOF) {
        putchar(c);  // Écho
    }
    
    // Lecture de nombres jusqu'à erreur
    int n;
    while (scanf("%d", &n) == 1) {
        printf("Lu : %d\n", n);
    }
    
    return 0;
}
```

### Parcours de tableau avec sentinelle

```c
#include <stdio.h>

int main(void) {
    // Tableau terminé par une valeur sentinelle
    int nombres[] = {5, 10, 15, 20, 25, -1};  // -1 = sentinelle
    
    int i = 0;
    while (nombres[i] != -1) {
        printf("%d ", nombres[i]);
        i++;
    }
    printf("\n");
    
    // Chaîne (sentinelle = '\0')
    char str[] = "Hello";
    for (int j = 0; str[j] != '\0'; j++) {
        printf("%c ", str[j]);
    }
    
    return 0;
}
```

### Algorithme de recherche

```c
#include <stdio.h>
#include <stdbool.h>

// Recherche linéaire
int recherche_lineaire(int arr[], int taille, int cible) {
    for (int i = 0; i < taille; i++) {
        if (arr[i] == cible) {
            return i;  // Trouvé
        }
    }
    return -1;  // Non trouvé
}

// Recherche binaire (tableau trié)
int recherche_binaire(int arr[], int taille, int cible) {
    int gauche = 0;
    int droite = taille - 1;
    
    while (gauche <= droite) {
        int milieu = gauche + (droite - gauche) / 2;  // Évite overflow
        
        if (arr[milieu] == cible) {
            return milieu;
        } else if (arr[milieu] < cible) {
            gauche = milieu + 1;
        } else {
            droite = milieu - 1;
        }
    }
    
    return -1;  // Non trouvé
}
```

### Validation d'entrée robuste

```c
#include <stdio.h>

int lire_entier_valide(const char *prompt, int min, int max) {
    int valeur;
    int items_lus;
    
    do {
        printf("%s [%d-%d] : ", prompt, min, max);
        items_lus = scanf("%d", &valeur);
        
        // Vider le buffer en cas d'entrée invalide
        while (getchar() != '\n');
        
        if (items_lus != 1) {
            printf("Erreur : entrez un nombre valide.\n");
        } else if (valeur < min || valeur > max) {
            printf("Erreur : hors limites.\n");
        }
    } while (items_lus != 1 || valeur < min || valeur > max);
    
    return valeur;
}

int main(void) {
    int age = lire_entier_valide("Votre âge", 0, 150);
    printf("Âge saisi : %d\n", age);
    return 0;
}
```

### Boucle avec timeout/limite

```c
#include <stdio.h>

#define MAX_TENTATIVES 3

int main(void) {
    int mot_de_passe_correct = 1234;
    int tentative;
    int essais_restants = MAX_TENTATIVES;
    
    while (essais_restants > 0) {
        printf("Mot de passe (%d essais restants) : ", essais_restants);
        scanf("%d", &tentative);
        
        if (tentative == mot_de_passe_correct) {
            printf("Accès autorisé!\n");
            break;
        }
        
        essais_restants--;
        if (essais_restants > 0) {
            printf("Incorrect, réessayez.\n");
        }
    }
    
    if (essais_restants == 0) {
        printf("Trop de tentatives, compte bloqué.\n");
    }
    
    return 0;
}
```

---

## Comparaison avec Python/JS/C#/Go

### Conditions

```c
// C
if (x > 0) {
    printf("Positif\n");
} else if (x < 0) {
    printf("Négatif\n");
} else {
    printf("Zéro\n");
}
```

```python
# Python : pas d'accolades, indentation obligatoire
if x > 0:
    print("Positif")
elif x < 0:  # elif, pas else if
    print("Négatif")
else:
    print("Zéro")
```

```javascript
// JavaScript : identique au C
if (x > 0) {
    console.log("Positif");
} else if (x < 0) {
    console.log("Négatif");
} else {
    console.log("Zéro");
}
```

### Boucles for

```c
// C : for classique avec index
for (int i = 0; i < n; i++) {
    printf("%d ", arr[i]);
}
```

```python
# Python : for-each (itération)
for item in arr:
    print(item)

# Avec index
for i, item in enumerate(arr):
    print(i, item)

# Range
for i in range(n):
    print(i)
```

```javascript
// JavaScript : plusieurs options
for (let i = 0; i < arr.length; i++) {
    console.log(arr[i]);
}

// for-of (valeurs)
for (const item of arr) {
    console.log(item);
}

// for-in (indices/clés) - à éviter pour les tableaux
for (const i in arr) {
    console.log(arr[i]);
}

// forEach
arr.forEach(item => console.log(item));
```

```go
// Go : for uniquement (remplace while aussi)
for i := 0; i < n; i++ {
    fmt.Println(i)
}

// range pour itération
for i, item := range arr {
    fmt.Println(i, item)
}

// while équivalent
for condition {
    // ...
}

// boucle infinie
for {
    // ...
}
```

### Switch

```c
// C : switch avec break obligatoire
switch (x) {
    case 1:
        printf("Un\n");
        break;
    case 2:
        printf("Deux\n");
        break;
    default:
        printf("Autre\n");
}
```

```python
# Python : match (3.10+), pas de fall-through
match x:
    case 1:
        print("Un")
    case 2:
        print("Deux")
    case _:
        print("Autre")
```

```go
// Go : switch sans break nécessaire (pas de fall-through par défaut)
switch x {
case 1:
    fmt.Println("Un")
case 2:
    fmt.Println("Deux")
default:
    fmt.Println("Autre")
}

// fallthrough explicite si besoin
switch x {
case 1:
    fmt.Println("Un")
    fallthrough  // Continue au case suivant
case 2:
    fmt.Println("Deux")
}
```

### Tableau comparatif

| Fonctionnalité          | C                 | Python        | JavaScript         | Go                |
|-------------------------|-------------------|---------------|--------------------|-------------------|
| if/else                 | ✅                | ✅ (elif)    | ✅                 | ✅               |
| switch                  | ✅ (break requis) | match (3.10+) | ✅ (break requis) | ✅ (pas de break)|
| for classique           | ✅                | ❌ (range)   | ✅                 | ✅               |
| for-each                | ❌                | for-in        | for-of             | for-range        |
| while                   | ✅                | ✅           | ✅                 | for              |
| do-while                | ✅                | ❌           | ✅                 | ❌               |
| break/continue          | ✅                | ✅           | ✅                 | ✅               |
| goto                    | ✅                | ❌           | ❌                 | ✅               |
| Labels (break/continue) | Via goto          | ❌           | ✅                 | ✅               |

---

## Exercices du chapitre

### Exercice 4.1 : Classification (Débutant)

Écrivez un programme qui demande un nombre et affiche :
- "Positif" si > 0
- "Négatif" si < 0
- "Zéro" si = 0
- "Pair" ou "Impair" (en plus)

---

### Exercice 4.2 : Calculatrice switch (Débutant)

Créez une calculatrice utilisant switch :
1. Demandez deux nombres
2. Demandez l'opération (+, -, *, /, %)
3. Affichez le résultat
4. Gérez la division par zéro et les opérateurs invalides

---

### Exercice 4.3 : FizzBuzz (Débutant)

Affichez les nombres de 1 à 100, mais :
- "Fizz" si divisible par 3
- "Buzz" si divisible par 5
- "FizzBuzz" si divisible par les deux

---

### Exercice 4.4 : Devine le nombre (Intermédiaire)

Créez un jeu où :
1. L'ordinateur choisit un nombre entre 1 et 100
2. L'utilisateur essaie de deviner
3. L'ordinateur dit "Plus grand" ou "Plus petit"
4. Comptez le nombre d'essais
5. Limitez à 7 essais maximum

Utilisez `rand()` de `<stdlib.h>` et `srand(time(NULL))` de `<time.h>`.

---

### Exercice 4.5 : Motifs géométriques (Intermédiaire)

Créez des fonctions pour afficher :

1. Un rectangle de `*` (largeur × hauteur)
2. Un triangle rectangle
3. Un triangle isocèle centré
4. Un losange

```
Rectangle 5x3:    Triangle:    Isocèle:    Losange:
*****             *               *            *
*****             **             ***          ***
*****             ***           *****        *****
                  ****         *******      *******
                  *****       *********      *****
                                              ***
                                               *
```

---

### Exercice 4.6 : Factoriel et Fibonacci (Intermédiaire)

Implémentez avec des boucles :
1. `factorial(n)` : n!
2. `fibonacci(n)` : nième nombre de Fibonacci
3. Affichez les 20 premiers termes de chaque séquence

---

### Exercice 4.7 : Tri à bulles (Intermédiaire)

Implémentez le tri à bulles (bubble sort) :
1. Parcourez le tableau
2. Échangez les éléments adjacents dans le mauvais ordre
3. Répétez jusqu'à ce qu'aucun échange ne soit nécessaire
4. Optimisez en détectant si le tableau est déjà trié

---

### Exercice 4.8 : Menu interactif (Intermédiaire)

Créez un programme avec un menu :
```
=== GESTION DE NOTES ===
1. Ajouter une note
2. Afficher les notes
3. Calculer la moyenne
4. Trouver min/max
5. Quitter
```

Utilisez un tableau pour stocker jusqu'à 100 notes.

---

### Exercice 4.9 : Nombre premier (Intermédiaire-Avancé)

Implémentez :
1. `est_premier(n)` : retourne 1 si n est premier, 0 sinon
2. `prochainPremier(n)` : retourne le premier nombre premier ≥ n
3. Affichez tous les nombres premiers entre 1 et 1000
4. Optimisez (ne tester que jusqu'à √n, sauter les pairs)

---

### Exercice 4.10 : Conversion de bases (Avancé)

Créez des fonctions pour convertir un entier :
1. Décimal → Binaire (chaîne)
2. Décimal → Hexadécimal (chaîne)
3. Binaire (chaîne) → Décimal
4. Hexadécimal (chaîne) → Décimal

Sans utiliser les formats %x ou %b de printf/scanf.

---

## Résumé du chapitre

✅ **if/else/else if** : Sélection conditionnelle, attention au `=` vs `==`

✅ **switch** : Multi-sélection sur constantes entières, `break` obligatoire

✅ **while** : Répétition tant que condition vraie, peut ne jamais s'exécuter

✅ **do-while** : Comme while mais s'exécute au moins une fois

✅ **for** : Boucle avec initialisation/condition/mise à jour intégrées

✅ **break** : Sort de la boucle/switch le plus proche

✅ **continue** : Passe à l'itération suivante

✅ **goto** : Saut inconditionnel, utile pour sortie multiple ou cleanup

✅ **Boucles imbriquées** : Attention à la complexité O(n²), O(n³)...

✅ **Idiomes** : Lecture EOF, sentinelles, validation d'entrée

---

**Prochain chapitre** : Fonctions et portée des variables
