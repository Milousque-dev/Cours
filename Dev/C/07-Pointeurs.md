# Chapitre 7 : Pointeurs

## Table des matières

1. [Concept de pointeur](#concept-de-pointeur)
2. [Déclaration et initialisation](#déclaration-et-initialisation)
3. [Opérateurs de pointeurs](#opérateurs-de-pointeurs)
4. [Arithmétique des pointeurs](#arithmétique-des-pointeurs)
5. [Pointeurs et tableaux](#pointeurs-et-tableaux)
6. [Pointeurs et fonctions](#pointeurs-et-fonctions)
7. [Pointeurs de pointeurs](#pointeurs-de-pointeurs)
8. [Pointeurs vers fonctions](#pointeurs-vers-fonctions)
9. [Pointeurs génériques (void *)](#pointeurs-génériques-void-)
10. [Erreurs courantes et débogage](#erreurs-courantes-et-débogage)
11. [Comparaison avec Python/JS/C#/Go](#comparaison-avec-pythonjscgo)
12. [Exercices du chapitre](#exercices-du-chapitre)

---

## Concept de pointeur

### Qu'est-ce qu'un pointeur ?

Un **pointeur** est une variable qui stocke l'**adresse mémoire** d'une autre variable, pas sa valeur directement.

```
Mémoire RAM :
┌──────────────────────────────────────────────┐
│  Adresse    │  Contenu     │  Variable       │
├─────────────┼──────────────┼─────────────────┤
│  0x1000     │  42          │  int x          │
│  0x1004     │  0x1000      │  int *ptr       │  ← ptr contient l'adresse de x
│  0x1008     │  ...         │  ...            │
└──────────────────────────────────────────────┘

ptr "pointe vers" x
```

### Pourquoi utiliser des pointeurs ?

1. **Passage par référence** : Modifier des variables dans une fonction
2. **Allocation dynamique** : Gérer la mémoire manuellement (malloc/free)
3. **Structures de données** : Listes chaînées, arbres, graphes
4. **Performance** : Éviter les copies de grandes structures
5. **Interopérabilité** : Accès au matériel, bibliothèques C

---

## Déclaration et initialisation

```c
#include <stdio.h>

int main(void) {
    int x = 42;
    
    // Déclaration d'un pointeur vers int
    int *ptr;      // ptr peut contenir l'adresse d'un int
    
    // Initialisation avec l'adresse de x
    ptr = &x;      // & = opérateur "adresse de"
    
    // Déclaration + initialisation en une ligne
    int *ptr2 = &x;
    
    // Pointeur nul (ne pointe vers rien)
    int *ptr_null = NULL;  // ou = 0
    
    // ATTENTION à la syntaxe de déclaration multiple
    int *p1, p2;    // p1 est int*, mais p2 est int (pas int*)!
    int *p3, *p4;   // p3 et p4 sont tous deux int*
    
    // Pointeurs vers différents types
    char c = 'A';
    char *pc = &c;
    
    double d = 3.14;
    double *pd = &d;
    
    return 0;
}
```

---

## Opérateurs de pointeurs

### Opérateur d'adresse `&`

Retourne l'adresse mémoire d'une variable :

```c
int x = 42;
int *ptr = &x;  // ptr contient l'adresse de x

printf("Valeur de x : %d\n", x);           // 42
printf("Adresse de x : %p\n", (void*)&x);  // 0x7ffd... (exemple)
printf("Valeur de ptr : %p\n", (void*)ptr); // Même adresse
```

### Opérateur de déréférencement `*`

Accède à la valeur pointée :

```c
int x = 42;
int *ptr = &x;

printf("Valeur pointée : %d\n", *ptr);  // 42 (déréférence ptr)

*ptr = 100;  // Modifie x via le pointeur!
printf("x après modification : %d\n", x);  // 100
```

### Résumé visuel

```
int x = 42;
int *ptr = &x;

    x                    ptr
  ┌─────┐              ┌─────────┐
  │  42 │ ←─────────── │ 0x1000  │
  └─────┘              └─────────┘
  0x1000                 0x1004

  &x   = 0x1000   (adresse de x)
  ptr  = 0x1000   (contenu de ptr = adresse)
  *ptr = 42       (valeur à l'adresse 0x1000)
```

---

## Arithmétique des pointeurs

L'arithmétique des pointeurs tient compte de la **taille du type pointé** :

```c
#include <stdio.h>

int main(void) {
    int arr[] = {10, 20, 30, 40, 50};
    int *p = arr;  // Pointe vers arr[0]
    
    printf("p     = %p, *p = %d\n", (void*)p, *p);      // arr[0] = 10
    printf("p + 1 = %p, *(p+1) = %d\n", (void*)(p+1), *(p+1)); // arr[1] = 20
    printf("p + 2 = %p, *(p+2) = %d\n", (void*)(p+2), *(p+2)); // arr[2] = 30
    
    // p + 1 ajoute sizeof(int) octets (généralement 4)
    // Si p = 0x1000, p + 1 = 0x1004 (pas 0x1001!)
    
    // Incrémentation
    p++;  // p pointe maintenant vers arr[1]
    printf("Après p++ : *p = %d\n", *p);  // 20
    
    // Déréférencement avec incrémentation
    int *q = arr;
    printf("*q++ = %d\n", *q++);   // Affiche 10, puis q avance
    printf("*++q = %d\n", *++q);   // q avance, puis affiche 30
    printf("++*q = %d\n", ++*q);   // Incrémente arr[2], affiche 31
    
    // Différence de pointeurs
    int *debut = arr;
    int *fin = &arr[4];
    printf("Différence : %td éléments\n", fin - debut);  // 4
    
    return 0;
}
```

### Opérations valides et invalides

```c
int *p1, *p2;
int n;

// VALIDE
p1 + n      // Avancer de n éléments
p1 - n      // Reculer de n éléments
p1 - p2     // Différence (nombre d'éléments entre les deux)
p1 < p2     // Comparaison (dans le même tableau)
p1 == p2    // Égalité
p1 == NULL  // Test de nullité

// INVALIDE (comportement indéfini ou erreur)
p1 + p2     // Addition de deux pointeurs
p1 * n      // Multiplication
p1 / n      // Division
```

---

## Pointeurs et tableaux

### Équivalence tableau/pointeur

```c
#include <stdio.h>

int main(void) {
    int arr[5] = {10, 20, 30, 40, 50};
    int *p = arr;  // arr décroît en pointeur vers arr[0]
    
    // Toutes ces expressions sont équivalentes :
    printf("arr[2] = %d\n", arr[2]);       // 30
    printf("*(arr + 2) = %d\n", *(arr + 2)); // 30
    printf("*(p + 2) = %d\n", *(p + 2));    // 30
    printf("p[2] = %d\n", p[2]);           // 30 (syntaxe tableau sur pointeur!)
    printf("2[arr] = %d\n", 2[arr]);       // 30 (valide mais bizarre!)
    
    // DIFFÉRENCE IMPORTANTE :
    // arr est un pointeur CONSTANT, on ne peut pas le modifier
    // arr++;  // ERREUR de compilation!
    
    // p est une variable pointeur, on peut le modifier
    p++;  // OK, p pointe maintenant vers arr[1]
    
    // sizeof différent!
    printf("sizeof(arr) = %zu\n", sizeof(arr));  // 20 (5 * 4 octets)
    printf("sizeof(p) = %zu\n", sizeof(p));      // 8 (taille d'un pointeur)
    
    return 0;
}
```

### Parcours de tableau avec pointeur

```c
#include <stdio.h>

void afficher_tableau(int *arr, int taille) {
    // Méthode 1 : indexation
    for (int i = 0; i < taille; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
    
    // Méthode 2 : pointeur avec incrémentation
    int *p = arr;
    for (int i = 0; i < taille; i++) {
        printf("%d ", *p++);
    }
    printf("\n");
    
    // Méthode 3 : pointeur avec comparaison
    int *fin = arr + taille;
    for (int *ptr = arr; ptr < fin; ptr++) {
        printf("%d ", *ptr);
    }
    printf("\n");
}
```

---

## Pointeurs et fonctions

### Passage par référence

```c
#include <stdio.h>

// Passage par valeur : ne modifie pas l'original
void incrementer_valeur(int n) {
    n++;  // Modifie la copie locale seulement
}

// Passage par référence (via pointeur) : modifie l'original
void incrementer_reference(int *n) {
    (*n)++;  // Modifie la valeur pointée
}

// Échanger deux variables
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int main(void) {
    int x = 10;
    
    incrementer_valeur(x);
    printf("Après incrementer_valeur : x = %d\n", x);  // 10 (pas changé)
    
    incrementer_reference(&x);
    printf("Après incrementer_reference : x = %d\n", x);  // 11
    
    int a = 5, b = 10;
    swap(&a, &b);
    printf("Après swap : a = %d, b = %d\n", a, b);  // a = 10, b = 5
    
    return 0;
}
```

### Retourner plusieurs valeurs

```c
#include <stdio.h>
#include <stdbool.h>

// Retourne quotient et reste via pointeurs
bool division(int a, int b, int *quotient, int *reste) {
    if (b == 0) {
        return false;  // Erreur
    }
    *quotient = a / b;
    *reste = a % b;
    return true;
}

// Trouver min et max
void minmax(int *arr, int n, int *min, int *max) {
    *min = *max = arr[0];
    for (int i = 1; i < n; i++) {
        if (arr[i] < *min) *min = arr[i];
        if (arr[i] > *max) *max = arr[i];
    }
}

int main(void) {
    int q, r;
    if (division(17, 5, &q, &r)) {
        printf("17 / 5 = %d reste %d\n", q, r);
    }
    
    int arr[] = {3, 7, 2, 9, 1, 5};
    int minimum, maximum;
    minmax(arr, 6, &minimum, &maximum);
    printf("Min: %d, Max: %d\n", minimum, maximum);
    
    return 0;
}
```

### Retourner un pointeur (attention!)

```c
#include <stdio.h>
#include <stdlib.h>

// MAUVAIS : retourne un pointeur vers une variable locale
int *mauvaise_fonction(void) {
    int local = 42;
    return &local;  // DANGER! local n'existera plus après return
}

// BON : retourne un pointeur vers de la mémoire allouée dynamiquement
int *creer_tableau(int taille) {
    int *arr = malloc(taille * sizeof(int));
    // L'appelant doit faire free(arr) plus tard!
    return arr;
}

// BON : retourne un pointeur vers une variable static
int *compteur(void) {
    static int count = 0;
    count++;
    return &count;  // OK car static a une durée de vie permanente
}
```

---

## Pointeurs de pointeurs

Un pointeur peut pointer vers un autre pointeur :

```c
#include <stdio.h>

int main(void) {
    int x = 42;
    int *p = &x;       // p pointe vers x
    int **pp = &p;     // pp pointe vers p
    int ***ppp = &pp;  // et ainsi de suite...
    
    printf("x = %d\n", x);
    printf("*p = %d\n", *p);
    printf("**pp = %d\n", **pp);
    printf("***ppp = %d\n", ***ppp);
    
    // Modification via pointeur de pointeur
    **pp = 100;
    printf("x après modification : %d\n", x);  // 100
    
    // Utilité : modifier un pointeur dans une fonction
    return 0;
}
```

### Application : modifier un pointeur dans une fonction

```c
#include <stdio.h>
#include <stdlib.h>

// Pour modifier un pointeur, on passe un pointeur vers ce pointeur
void allouer_tableau(int **ptr, int taille) {
    *ptr = malloc(taille * sizeof(int));
}

void liberer_et_nullifier(int **ptr) {
    free(*ptr);
    *ptr = NULL;  // Empêche use-after-free
}

int main(void) {
    int *tableau = NULL;
    
    allouer_tableau(&tableau, 10);  // Passe l'adresse du pointeur
    if (tableau != NULL) {
        for (int i = 0; i < 10; i++) {
            tableau[i] = i * 10;
        }
    }
    
    liberer_et_nullifier(&tableau);
    // tableau est maintenant NULL
    
    return 0;
}
```

### Tableaux de chaînes (char **)

```c
#include <stdio.h>

int main(void) {
    // Tableau de pointeurs vers chaînes
    const char *jours[] = {"Lun", "Mar", "Mer", "Jeu", "Ven", "Sam", "Dim"};
    
    // char ** pointe vers le premier élément
    const char **p = jours;
    
    for (int i = 0; i < 7; i++) {
        printf("%s ", *p++);  // ou jours[i] ou p[i]
    }
    printf("\n");
    
    // main reçoit argc et argv
    // int main(int argc, char **argv)
    // argv[0] = nom du programme
    // argv[1], argv[2]... = arguments
    
    return 0;
}
```

---

## Pointeurs vers fonctions

```c
#include <stdio.h>

// Fonctions exemples
int addition(int a, int b) { return a + b; }
int soustraction(int a, int b) { return a - b; }
int multiplication(int a, int b) { return a * b; }

int main(void) {
    // Déclaration d'un pointeur vers fonction
    int (*operation)(int, int);
    
    // Affectation
    operation = addition;       // ou &addition
    printf("5 + 3 = %d\n", operation(5, 3));
    
    operation = multiplication;
    printf("5 * 3 = %d\n", operation(5, 3));
    
    // Tableau de pointeurs de fonctions
    int (*operations[])(int, int) = {addition, soustraction, multiplication};
    char *symboles[] = {"+", "-", "*"};
    
    for (int i = 0; i < 3; i++) {
        printf("10 %s 4 = %d\n", symboles[i], operations[i](10, 4));
    }
    
    return 0;
}
```

### Callbacks

```c
#include <stdio.h>
#include <stdlib.h>

// Fonction callback
typedef int (*Comparateur)(const void*, const void*);

int comparer_croissant(const void *a, const void *b) {
    return (*(int*)a - *(int*)b);
}

int comparer_decroissant(const void *a, const void *b) {
    return (*(int*)b - *(int*)a);
}

int main(void) {
    int arr[] = {5, 2, 8, 1, 9, 3};
    int n = sizeof(arr) / sizeof(arr[0]);
    
    // qsort utilise un callback pour la comparaison
    qsort(arr, n, sizeof(int), comparer_croissant);
    
    printf("Trié croissant : ");
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    printf("\n");
    
    qsort(arr, n, sizeof(int), comparer_decroissant);
    
    printf("Trié décroissant : ");
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    printf("\n");
    
    return 0;
}
```

---

## Pointeurs génériques (void *)

`void *` peut pointer vers n'importe quel type, mais ne peut pas être déréférencé directement :

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void afficher_memoire(void *ptr, size_t taille) {
    unsigned char *bytes = (unsigned char *)ptr;
    for (size_t i = 0; i < taille; i++) {
        printf("%02X ", bytes[i]);
    }
    printf("\n");
}

int main(void) {
    int x = 0x12345678;
    double d = 3.14;
    char str[] = "ABC";
    
    printf("int : ");
    afficher_memoire(&x, sizeof(x));
    
    printf("double : ");
    afficher_memoire(&d, sizeof(d));
    
    printf("string : ");
    afficher_memoire(str, strlen(str) + 1);
    
    // malloc retourne void*
    void *generic = malloc(100);
    
    // Cast nécessaire pour l'utiliser
    int *arr = (int *)generic;
    for (int i = 0; i < 25; i++) {
        arr[i] = i;
    }
    
    free(generic);
    
    return 0;
}
```

---

## Erreurs courantes et débogage

### 1. Pointeur non initialisé

```c
int *p;        // Contient une adresse aléatoire (garbage)
*p = 42;       // CRASH! Écrit dans une zone mémoire aléatoire

// Solution : toujours initialiser
int *p = NULL;
// ou
int x;
int *p = &x;
```

### 2. Déréférencement de NULL

```c
int *p = NULL;
*p = 42;  // CRASH! Segmentation fault

// Solution : vérifier avant déréférencement
if (p != NULL) {
    *p = 42;
}
```

### 3. Use-after-free

```c
int *p = malloc(sizeof(int));
*p = 42;
free(p);
*p = 100;  // CRASH ou corruption! Mémoire libérée

// Solution : mettre à NULL après free
free(p);
p = NULL;
```

### 4. Double free

```c
int *p = malloc(sizeof(int));
free(p);
free(p);  // CRASH! Double libération

// Solution : mettre à NULL après free
free(p);
p = NULL;
// free(NULL) est sûr (ne fait rien)
```

### 5. Fuite mémoire (memory leak)

```c
void fuite(void) {
    int *p = malloc(100 * sizeof(int));
    // Oubli de free(p)!
    // La mémoire est perdue quand la fonction retourne
}

// Solution : toujours free ce qui est malloc
```

### 6. Retour de pointeur local

```c
int *mauvais(void) {
    int local = 42;
    return &local;  // local n'existe plus après return!
}

// Solution : utiliser malloc ou static
```

### Outils de débogage

```bash
# Valgrind : détecte les fuites et erreurs mémoire
valgrind --leak-check=full ./programme

# AddressSanitizer (compilation)
gcc -fsanitize=address -g programme.c -o programme
./programme
```

---

## Comparaison avec Python/JS/C#/Go

| Aspect                 | C         | Python    | JavaScript | Go        |
|------------------------|-----------|-----------|------------|-----------|
| Pointeurs explicites   | ✅ Oui   | ❌ Non    | ❌ Non    | ✅ Oui    |
| Arithmétique pointeurs | ✅ Oui   | ❌ Non    | ❌ Non    | ❌ Non    |
| Références             | Pointeurs | Implicite | Implicite  | Pointeurs |
| Gestion mémoire        | Manuelle  | GC        | GC         | GC        |
| Pointeur nul           | `NULL`    | `None`    | `null`     | `nil`     |

```go
// Go : pointeurs similaires mais pas d'arithmétique
x := 42
p := &x       // p est un *int
fmt.Println(*p) // 42
// p++  // ERREUR : pas d'arithmétique
```

```python
# Python : tout est référence, pas de pointeurs explicites
x = 42
# Impossible d'avoir un "pointeur vers x"
# Les objets mutables se comportent comme des références
```

---

## Exercices du chapitre

### Exercice 7.1 : Manipulation de base (Débutant)
Créez des fonctions utilisant des pointeurs pour :
- `void echanger(int *a, int *b)`
- `void tripler(int *n)`
- `void mettre_a_zero(int *n)`

### Exercice 7.2 : Tableaux et pointeurs (Débutant)
Écrivez des fonctions de manipulation de tableaux utilisant l'arithmétique des pointeurs :
- `int *trouver(int *arr, int n, int val)` : retourne pointeur vers élément
- `void inverser(int *arr, int n)` : inverse le tableau en place

### Exercice 7.3 : Chaînes avec pointeurs (Intermédiaire)
Réimplémentez avec des pointeurs :
- `size_t strlen_ptr(const char *s)`
- `char *strcpy_ptr(char *dest, const char *src)`
- `int strcmp_ptr(const char *s1, const char *s2)`

### Exercice 7.4 : Allocation dynamique (Intermédiaire)
Créez des fonctions :
- `int *creer_sequence(int debut, int fin)` : crée [debut, debut+1, ..., fin]
- `int *concatener(int *a, int na, int *b, int nb)` : fusionne deux tableaux

### Exercice 7.5 : Matrice dynamique (Avancé)
Implémentez une matrice 2D allouée dynamiquement :
- `int **creer_matrice(int lignes, int colonnes)`
- `void liberer_matrice(int **matrice, int lignes)`
- `void remplir_identite(int **matrice, int n)`

### Exercice 7.6 : Liste chaînée (Avancé)
Créez une liste chaînée simple avec :
- Structure `Node` avec `int data` et `Node *next`
- `void inserer_debut(Node **tete, int val)`
- `void inserer_fin(Node **tete, int val)`
- `void supprimer(Node **tete, int val)`
- `void afficher(Node *tete)`
- `void liberer_liste(Node **tete)`

---

## Résumé du chapitre

**Pointeur** : Variable contenant une adresse mémoire

**Opérateurs** : `&` (adresse), `*` (déréférencement)

**Arithmétique** : `p + n` avance de `n * sizeof(*p)` octets

**Tableau ≈ pointeur** : `arr[i]` équivaut à `*(arr + i)`

**Passage par référence** : Passer `&var` et recevoir `type *param`

**Pointeurs de pointeurs** : Pour modifier un pointeur dans une fonction

**void*** : Pointeur générique, nécessite cast pour utilisation

**Erreurs courantes** : Non-initialisé, NULL, use-after-free, double free, fuites

---

**Prochain chapitre** : Structures, unions et énumérations
