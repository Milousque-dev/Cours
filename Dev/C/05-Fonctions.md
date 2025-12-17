# Chapitre 5 : Fonctions et portée des variables

## Table des matières

1.  [Introduction aux fonctions](#introduction-aux-fonctions)
2.  [Déclaration et définition](#déclaration-et-définition)
3.  [Paramètres et arguments](#paramètres-et-arguments)
4.  [Valeurs de retour](#valeurs-de-retour)
5.  [Passage par valeur vs passage par référence](#passage-par-valeur-vs-passage-par-référence)
6.  [Portée des variables (scope)](#portée-des-variables-scope)
7.  [Classes de stockage](#classes-de-stockage)
8.  [Récursivité](#récursivité)
9.  [Pointeurs de fonctions](#pointeurs-de-fonctions)
10. [Fonctions inline et macros](#fonctions-inline-et-macros)
11. [Comparaison avec Python/JS/C#/Go](#comparaison-avec-pythonjscgo)
12. [Exercices du chapitre](#exercices-du-chapitre)

---

## Introduction aux fonctions

### Qu'est-ce qu'une fonction ?

Une fonction est un **bloc de code réutilisable** qui effectue une tâche spécifique. Elle permet de :

- **Modulariser** le code (diviser en parties gérables)
- **Réutiliser** du code (éviter la duplication)
- **Abstraire** la complexité (cacher les détails d'implémentation)

```c
#include <stdio.h>

// Définition d'une fonction
int addition(int a, int b) {
    return a + b;
}

void afficher_bonjour(void) {
    printf("Bonjour!\n");
}

int main(void) {
    afficher_bonjour();
    int resultat = addition(5, 3);
    printf("5 + 3 = %d\n", resultat);
    return 0;
}
```

---

## Déclaration et définition

### Prototypes

```c
#include <stdio.h>

// DÉCLARATIONS (prototypes) - en haut du fichier
int calculer_carre(int n);
double calculer_moyenne(double *tableau, int taille);

int main(void) {
    printf("5² = %d\n", calculer_carre(5));
    return 0;
}

// DÉFINITIONS - après main() grâce aux prototypes
int calculer_carre(int n) {
    return n * n;
}

double calculer_moyenne(double *tableau, int taille) {
    double somme = 0.0;
    for (int i = 0; i < taille; i++) {
        somme += tableau[i];
    }
    return somme / taille;
}
```

---

## Paramètres et arguments

```c
// Paramètres de base
int somme(int a, int b) {
    return a + b;
}

// Tableau (décroît en pointeur)
void afficher_tableau(int arr[], int taille) {
    for (int i = 0; i < taille; i++) {
        printf("%d ", arr[i]);
    }
}

// Pointeur pour modification
void incrementer(int *valeur) {
    (*valeur)++;
}

// Paramètre const (lecture seule)
int longueur_chaine(const char *str) {
    int len = 0;
    while (str[len]) len++;
    return len;
}
```

---

## Valeurs de retour

```c
// Retour simple
int carre(int n) {
    return n * n;
}

// Retour multiple via pointeurs
bool diviser(int a, int b, int *quotient, int *reste) {
    if (b == 0) return false;
    *quotient = a / b;
    *reste = a % b;
    return true;
}

// Retour structure
struct Resultat {
    int quotient;
    int reste;
};

struct Resultat division_euclidienne(int a, int b) {
    struct Resultat r = {a / b, a % b};
    return r;
}
```

---

## Passage par valeur vs passage par référence

```c
// Passage par VALEUR : copie locale
void doubler_valeur(int n) {
    n = n * 2;  // Modifie la copie seulement
}

// Passage par RÉFÉRENCE via pointeur
void doubler_reference(int *n) {
    *n = (*n) * 2;  // Modifie l'original
}

// Exemple swap
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int main(void) {
    int x = 5, y = 10;
    swap(&x, &y);  // x=10, y=5
    return 0;
}
```

---

## Portée des variables (scope)

```c
int globale = 100;  // Portée globale

void fonction(void) {
    int locale = 50;  // Portée locale
    printf("%d %d\n", globale, locale);
}

int main(void) {
    int main_var = 10;
    {
        int bloc_var = 25;  // Portée bloc
    }
    // bloc_var n'existe plus ici
    return 0;
}
```

---

## Classes de stockage

```c
// static locale : conserve sa valeur entre appels
void compteur(void) {
    static int count = 0;
    count++;
    printf("Appel %d\n", count);
}

// static globale : visible uniquement dans ce fichier
static int variable_privee = 42;

// extern : déclaration d'une variable définie ailleurs
extern int variable_externe;
```

---

## Récursivité

```c
// Factorielle
unsigned long factorielle(int n) {
    if (n <= 1) return 1;
    return n * factorielle(n - 1);
}

// Fibonacci
int fibonacci(int n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

// PGCD (Euclide)
int pgcd(int a, int b) {
    if (b == 0) return a;
    return pgcd(b, a % b);
}
```

---

## Pointeurs de fonctions

```c
#include <stdio.h>

int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }
int mul(int a, int b) { return a * b; }

int main(void) {
    // Déclaration
    int (*operation)(int, int);
    
    operation = add;
    printf("5 + 3 = %d\n", operation(5, 3));
    
    operation = mul;
    printf("5 * 3 = %d\n", operation(5, 3));
    
    // Tableau de pointeurs de fonctions
    int (*ops[])(int, int) = {add, sub, mul};
    for (int i = 0; i < 3; i++) {
        printf("Résultat %d : %d\n", i, ops[i](10, 5));
    }
    
    return 0;
}
```

---

## Fonctions inline et macros

```c
// Fonction inline (C99+)
static inline int max(int a, int b) {
    return (a > b) ? a : b;
}

// Macro (attention aux pièges!)
#define CARRE(x) ((x) * (x))
#define MAX(a, b) ((a) > (b) ? (a) : (b))

// PIÈGE : effets de bord
int x = 5;
int y = CARRE(x++);  // x++ évalué DEUX fois!
```

---

## Comparaison avec Python/JS/C#/Go

| Aspect             | C                | Python          | JavaScript       | Go       |
|--------------------|------------------|-----------------|------------------|----------|
| Passage défaut     | Valeur           | Référence objet | Valeur/Référence | Valeur   |
| Référence          | Pointeur         | Idem            | Idem             | Pointeur |
| Retours multiples  | Pointeurs/struct | Tuple natif     | Array/Object     | Natif    |
| Fonctions anonymes | Non              | Lambda          | Arrow functions  | Closures |

---

## Exercices du chapitre

### Exercice 5.1 : Fonctions mathématiques (Débutant)
Implémentez `puissance`, `factorielle`, `pgcd`, `ppcm`.

### Exercice 5.2 : Manipulation de tableaux (Débutant)
Créez `somme`, `moyenne`, `minimum`, `maximum`, `inverser`.

### Exercice 5.3 : Passage par référence (Intermédiaire)
Créez `swap` et `ordonner_trois` (met 3 entiers en ordre croissant).

### Exercice 5.4 : Récursivité (Intermédiaire)
Implémentez récursivement : somme 1 à n, nombre de chiffres, somme des chiffres.

### Exercice 5.5 : Pointeurs de fonctions (Avancé)
Créez une calculatrice avec un tableau de pointeurs de fonctions.

---

## Résumé du chapitre

**Fonctions** : Blocs de code réutilisables

**Prototypes** : Déclarent la signature avant utilisation

**Passage par valeur** : La fonction reçoit une copie

**Passage par référence** : Via pointeurs pour modifier l'original

**Static** : Locale persistante ou globale restreinte au fichier

**Récursivité** : Fonction qui s'appelle elle-même

**Pointeurs de fonctions** : Permettent callbacks et polymorphisme

---

**Prochain chapitre** : Tableaux et chaînes de caractères
