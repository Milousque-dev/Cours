# Chapitre 2 : Variables, types de données et constantes

## Table des matières

1. [Concept de variable en C](#concept-de-variable-en-c)
2. [Types de données fondamentaux](#types-de-données-fondamentaux)
3. [Déclaration et initialisation](#déclaration-et-initialisation)
4. [Modificateurs de types](#modificateurs-de-types)
5. [Types entiers de taille fixe (C99+)](#types-entiers-de-taille-fixe-c99)
6. [Constantes et littéraux](#constantes-et-littéraux)
7. [Portée et durée de vie des variables](#portée-et-durée-de-vie-des-variables)
8. [Qualificateurs de type](#qualificateurs-de-type)
9. [Conversions de types (casting)](#conversions-de-types-casting)
10. [Entrées/Sorties formatées](#entréessorties-formatées)
11. [Comparaison avec Python/JS/C#/Go](#comparaison-avec-pythonjscgo)
12. [Exercices du chapitre](#exercices-du-chapitre)

---

## Concept de variable en C

### Qu'est-ce qu'une variable ?

Une variable en C est un **emplacement mémoire nommé** qui stocke une valeur d'un type spécifique. Contrairement à Python ou JavaScript, en C :

- Le **type est fixe** et déterminé à la compilation
- La **taille mémoire** est connue et réservée à l'avance
- L'**adresse mémoire** est accessible via l'opérateur `&`

```c
int age = 25;
// Réserve 4 octets en mémoire
// Stocke la valeur 25 à cette adresse
// 'age' est un alias pour cette zone mémoire
```

Représentation en mémoire :

```
Adresse         Contenu (binaire)              Interprétation
0x7ffd1234      00000000 00000000 00000000 00011001   = 25 (int)
                ├─────────────────────────────────┤
                         4 octets (32 bits)
```

### Règles de nommage des variables

| Règle                           | Valide                         | Invalide              |
|---------------------------------|--------------------------------|-----------------------|
| Commence par lettre ou `_`      | `age`, `_count`, `valeur1`     | `1valeur`, `@prix`    |
| Contient lettres, chiffres, `_` | `total_prix`, `var2`           | `total-prix`, `var.2` |
| Sensible à la casse             | `Age` ≠ `age` ≠ `AGE`          | -                     |
| Pas un mot réservé              | `compteur`                     | `int`, `return`, `if` |
| Longueur illimitée              | `unNomTresLongMaisPeuPratique` | -                     |

### Mots réservés du C

```c
// Mots-clés C (à ne jamais utiliser comme identifiant)
auto      break     case      char      const     continue
default   do        double    else      enum      extern
float     for       goto      if        inline    int
long      register  restrict  return    short     signed
sizeof    static    struct    switch    typedef   union
unsigned  void      volatile  while     _Bool     _Complex
_Imaginary

// C99+ ajouts
_Alignas  _Alignof  _Atomic   _Generic  _Noreturn _Static_assert
_Thread_local

// C23 ajouts
true      false     nullptr   typeof    typeof_unqual
```

### Conventions de nommage

```c
// Style snake_case (recommandé en C)
int nombre_etudiants;
float prix_unitaire;
char nom_fichier[100];

// Style camelCase (courant aussi)
int nombreEtudiants;
float prixUnitaire;

// MAJUSCULES pour les constantes et macros
#define MAX_BUFFER_SIZE 1024
const int TAILLE_MAX = 100;

// Préfixe pour les variables globales (convention)
int g_compteur_global;

// Préfixe pour les pointeurs (convention)
int *p_valeur;
char *str_nom;
```

---

## Types de données fondamentaux

### Vue d'ensemble

Le C possède quatre catégories de types fondamentaux :

```
Types fondamentaux C
├── Entiers (integers)
│   ├── char      (1 octet)
│   ├── short     (≥2 octets)
│   ├── int       (≥2 octets, généralement 4)
│   ├── long      (≥4 octets)
│   └── long long (≥8 octets, C99+)
│
├── Nombres à virgule flottante
│   ├── float     (4 octets, ~7 chiffres significatifs)
│   ├── double    (8 octets, ~15 chiffres significatifs)
│   └── long double (≥8 octets, précision étendue)
│
├── Caractères
│   └── char      (1 octet, peut être signé ou non selon implémentation)
│
└── Vide
    └── void      (absence de type/valeur)
```

### Type `char` (1 octet)

Le type `char` stocke un caractère (en réalité, son code ASCII/Unicode) :

```c
char lettre = 'A';        // Code ASCII : 65
char chiffre = '7';       // Code ASCII : 55 (pas la valeur 7!)
char newline = '\n';      // Code ASCII : 10
char nul = '\0';          // Code ASCII : 0 (terminateur de chaîne)

// char est un petit entier (signé ou non selon compilateur)
char c = 65;              // Équivalent à 'A'
printf("%c = %d\n", c, c); // Affiche : A = 65

// Plage de valeurs
// signed char   : -128 à 127
// unsigned char : 0 à 255
```

### Types entiers

```c
#include <stdio.h>
#include <limits.h>  // Contient les limites des types

int main(void) {
    // short : au moins 16 bits
    short s = 32767;
    printf("short : %d octets, max = %d\n", sizeof(short), SHRT_MAX);
    
    // int : au moins 16 bits (généralement 32)
    int i = 2147483647;
    printf("int : %zu octets, max = %d\n", sizeof(int), INT_MAX);
    
    // long : au moins 32 bits
    long l = 2147483647L;  // Suffixe L pour littéral long
    printf("long : %zu octets, max = %ld\n", sizeof(long), LONG_MAX);
    
    // long long : au moins 64 bits (C99+)
    long long ll = 9223372036854775807LL;  // Suffixe LL
    printf("long long : %zu octets, max = %lld\n", sizeof(long long), LLONG_MAX);
    
    return 0;
}
```

Sortie typique sur système 64 bits :

```
short : 2 octets, max = 32767
int : 4 octets, max = 2147483647
long : 8 octets, max = 9223372036854775807
long long : 8 octets, max = 9223372036854775807
```

### Types à virgule flottante

```c
#include <stdio.h>
#include <float.h>  // Contient les limites des flottants

int main(void) {
    // float : simple précision (32 bits)
    float f = 3.14159f;  // Suffixe f obligatoire pour float
    printf("float : %zu octets, précision = %d chiffres\n", 
           sizeof(float), FLT_DIG);
    
    // double : double précision (64 bits) - type par défaut
    double d = 3.141592653589793;
    printf("double : %zu octets, précision = %d chiffres\n", 
           sizeof(double), DBL_DIG);
    
    // long double : précision étendue (≥64 bits, souvent 80 ou 128)
    long double ld = 3.14159265358979323846L;  // Suffixe L
    printf("long double : %zu octets, précision = %d chiffres\n", 
           sizeof(long double), LDBL_DIG);
    
    // Notation scientifique
    double avogadro = 6.022e23;   // 6.022 × 10²³
    double electron = 1.6e-19;   // 1.6 × 10⁻¹⁹
    
    return 0;
}
```

### Problèmes de précision des flottants

```c
#include <stdio.h>

int main(void) {
    float f = 0.1f;
    double d = 0.1;
    
    // 0.1 n'est pas représentable exactement en binaire
    printf("float  0.1 = %.20f\n", f);   // 0.10000000149011611938
    printf("double 0.1 = %.20lf\n", d);  // 0.10000000000000000555
    
    // Comparaison de flottants : JAMAIS avec ==
    float a = 0.1f + 0.2f;
    float b = 0.3f;
    
    if (a == b) {
        printf("Égaux\n");  // Probablement NON exécuté!
    } else {
        printf("Différents : %.20f vs %.20f\n", a, b);
    }
    
    // Bonne pratique : comparaison avec tolérance (epsilon)
    #define EPSILON 1e-7
    if (fabsf(a - b) < EPSILON) {
        printf("Approximativement égaux\n");
    }
    
    return 0;
}
```

### Type `void`

Le type `void` indique l'absence de type :

```c
// Fonction sans valeur de retour
void afficher_bonjour(void) {
    printf("Bonjour!\n");
    // pas de return valeur
}

// Fonction sans paramètres
int obtenir_valeur(void) {
    return 42;
}

// Pointeur générique (peut pointer sur n'importe quoi)
void *ptr_generique;
int x = 10;
ptr_generique = &x;  // Pointe sur un int
int *p = (int *)ptr_generique;  // Cast nécessaire pour utiliser
```

### Type `_Bool` / `bool` (C99+)

```c
#include <stdbool.h>  // Définit bool, true, false

int main(void) {
    bool actif = true;
    bool termine = false;
    
    if (actif && !termine) {
        printf("En cours...\n");
    }
    
    // _Bool est le type sous-jacent
    _Bool flag = 1;  // Équivalent à true
    
    // Toute valeur non-nulle est vraie
    bool test1 = 42;    // true
    bool test2 = -1;    // true
    bool test3 = 0;     // false
    bool test4 = 0.5;   // true (converti en 1)
    
    printf("sizeof(bool) = %zu\n", sizeof(bool));  // 1 octet
    
    return 0;
}
```

---

## Déclaration et initialisation

### Syntaxe de déclaration

```c
// Déclaration simple
type nom_variable;

// Déclaration avec initialisation
type nom_variable = valeur;

// Déclarations multiples
type var1, var2, var3;

// Déclarations multiples avec initialisations mixtes
type var1 = val1, var2, var3 = val3;
```

### Exemples

```c
// Déclarations sans initialisation (valeur indéterminée!)
int compteur;           // Contient des "déchets" mémoire
float temperature;      // DANGER: utiliser avant initialisation = bug
char caractere;

// Déclarations avec initialisation (recommandé)
int age = 25;
float prix = 19.99f;
char grade = 'A';
double pi = 3.14159265358979;

// Déclarations multiples (même type)
int x = 0, y = 0, z = 0;
int a, b, c;  // Non initialisées

// Attention aux pointeurs dans déclarations multiples!
int *p1, p2;    // p1 est int*, mais p2 est int (pas int*!)
int *p3, *p4;   // p3 et p4 sont tous deux int*
```

### Variables non initialisées : le danger

```c
#include <stdio.h>

int main(void) {
    int x;  // Non initialisée : contient des "déchets"
    
    printf("x = %d\n", x);  // COMPORTEMENT INDÉFINI!
    // Peut afficher 0, 42, -1234567, n'importe quoi...
    // Le programme peut crasher
    // Le compilateur peut optimiser de manière inattendue
    
    // Avec -Wall, gcc affiche un warning
    
    // Bonne pratique : toujours initialiser
    int y = 0;
    printf("y = %d\n", y);  // Toujours 0
    
    return 0;
}
```

### Initialisation désignée (C99+)

Pour les tableaux et structures :

```c
// Tableaux avec initialisateurs désignés
int tab[10] = {[0] = 1, [5] = 50, [9] = 100};
// tab = {1, 0, 0, 0, 0, 50, 0, 0, 0, 100}

// Structures avec initialisateurs désignés
struct Point {
    int x;
    int y;
};

struct Point p = {.y = 10, .x = 5};  // Ordre flexible
```

---

## Modificateurs de types

### `signed` et `unsigned`

Par défaut, les types entiers sont signés. Le modificateur `unsigned` double la plage positive en éliminant les valeurs négatives :

```c
#include <stdio.h>
#include <limits.h>

int main(void) {
    // signed (défaut pour int, short, long)
    signed int si = -100;       // -2147483648 à 2147483647
    int i = -100;               // Équivalent
    
    // unsigned
    unsigned int ui = 100;      // 0 à 4294967295
    unsigned int err = -1;      // ATTENTION: devient 4294967295!
    
    printf("signed int max   = %d\n", INT_MAX);       // 2147483647
    printf("unsigned int max = %u\n", UINT_MAX);      // 4294967295
    
    // char : le signe dépend de l'implémentation!
    char c = 128;               // Peut être -128 ou 128 selon compilateur
    signed char sc = -50;       // Toujours signé : -128 à 127
    unsigned char uc = 200;     // Toujours non signé : 0 à 255
    
    return 0;
}
```

### Tableau récapitulatif des plages

| Type                 | Octets | Minimum                    | Maximum                    |
|----------------------|--------|----------------------------|----------------------------|
| `char`               | 1      | -128 ou 0                  | 127 ou 255                 |
| `signed char`        | 1      | -128                       | 127                        |
| `unsigned char`      | 1      | 0                          | 255                        |
| `short`              | 2      | -32 768                    | 32 767                     |
| `unsigned short`     | 2      | 0                          | 65 535                     |
| `int`                | 4      | -2 147 483 648             | 2 147 483 647              |
| `unsigned int`       | 4      | 0                          | 4 294 967 295              |
| `long`               | 4 ou 8 | -2³¹ ou -2⁶³               | 2³¹-1 ou 2⁶³-1             |
| `unsigned long`      | 4 ou 8 | 0                          | 2³²-1 ou 2⁶⁴-1             |
| `long long`          | 8      | -9 223 372 036 854 775 808 | 9 223 372 036 854 775 807  |
| `unsigned long long` | 8      | 0                          | 18 446 744 073 709 551 615 |

### Piège du débordement (overflow)

```c
#include <stdio.h>
#include <limits.h>

int main(void) {
    // Débordement signed : comportement indéfini!
    int max_int = INT_MAX;
    printf("INT_MAX = %d\n", max_int);
    printf("INT_MAX + 1 = %d\n", max_int + 1);  // Indéfini! (souvent -2147483648)
    
    // Débordement unsigned : wrapping défini
    unsigned int max_uint = UINT_MAX;
    printf("UINT_MAX = %u\n", max_uint);
    printf("UINT_MAX + 1 = %u\n", max_uint + 1);  // Toujours 0
    
    // Piège classique : boucle infinie
    for (unsigned int i = 10; i >= 0; i--) {
        printf("%u ", i);  // Boucle infinie! 0 - 1 = UINT_MAX
        if (i == 0) break; // Solution
    }
    
    return 0;
}
```

---

## Types entiers de taille fixe (C99+)

Le header `<stdint.h>` définit des types à taille garantie :

```c
#include <stdint.h>
#include <inttypes.h>  // Pour les macros de format

int main(void) {
    // Types de taille exacte
    int8_t    i8  = -128;              // Exactement 8 bits signés
    int16_t   i16 = -32768;            // Exactement 16 bits signés
    int32_t   i32 = -2147483648;       // Exactement 32 bits signés
    int64_t   i64 = -9223372036854775807LL;  // Exactement 64 bits signés
    
    uint8_t   u8  = 255;               // Exactement 8 bits non signés
    uint16_t  u16 = 65535;             // Exactement 16 bits non signés
    uint32_t  u32 = 4294967295U;       // Exactement 32 bits non signés
    uint64_t  u64 = 18446744073709551615ULL;  // Exactement 64 bits non signés
    
    // Affichage avec macros portables
    printf("int32_t : %" PRId32 "\n", i32);
    printf("uint64_t : %" PRIu64 "\n", u64);
    
    // Types "au moins"
    int_least8_t   il8;   // Au moins 8 bits
    int_least32_t  il32;  // Au moins 32 bits
    
    // Types "rapides" (taille optimale pour la plateforme)
    int_fast8_t    if8;   // Au moins 8 bits, le plus rapide
    int_fast32_t   if32;  // Au moins 32 bits, le plus rapide
    
    // Type pour les tailles (toujours non signé)
    size_t taille = sizeof(int);  // Taille garantie suffisante
    printf("size_t : %zu\n", taille);
    
    // Type pour les différences de pointeurs
    ptrdiff_t diff;  // Signé
    
    // Type pour les pointeurs
    intptr_t  ptr_signe;     // Peut stocker un pointeur (signé)
    uintptr_t ptr_non_signe; // Peut stocker un pointeur (non signé)
    
    return 0;
}
```

### Quand utiliser quel type ?

| Usage                                | Type recommandé      |
|--------------------------------------|----------------------|
| Compteur de boucle                   | `int` ou `size_t`    |
| Taille de tableau                    | `size_t`             |
| Données binaires 8 bits              | `uint8_t`            |
| Protocole réseau 32 bits             | `uint32_t`           |
| Code portable sur toutes plateformes | `int32_t`, `int64_t` |
| Arithmétique rapide                  | `int_fast32_t`       |
| Index de tableau                     | `size_t`             |

---

## Constantes et littéraux

### Littéraux entiers

```c
int decimal     = 42;        // Décimal (base 10)
int octal       = 052;       // Octal (base 8) - préfixe 0
int hexadecimal = 0x2A;      // Hexadécimal (base 16) - préfixe 0x ou 0X
int binaire     = 0b101010;  // Binaire (C23, extensions GCC/Clang) - préfixe 0b

// Tous égaux à 42

// Suffixes pour le type
long l            = 42L;     // long
unsigned u        = 42U;     // unsigned int
unsigned long ul  = 42UL;    // unsigned long
long long ll      = 42LL;    // long long
unsigned long long ull = 42ULL;  // unsigned long long
```

### Littéraux flottants

```c
float f1 = 3.14f;           // float (suffixe f obligatoire)
float f2 = 3.14F;           // float (F majuscule aussi valide)
double d1 = 3.14;           // double (par défaut)
double d2 = 3.14e10;        // Notation scientifique : 3.14 × 10¹⁰
double d3 = 1.5e-5;         // 1.5 × 10⁻⁵ = 0.000015
long double ld = 3.14L;     // long double

// Hexadécimal flottant (C99+)
double hex_float = 0x1.fp3;  // 1.9375 × 2³ = 15.5
```

### Littéraux caractères

```c
char c1 = 'A';              // Caractère ASCII : 65
char c2 = '\n';             // Séquence d'échappement : nouvelle ligne
char c3 = '\t';             // Tabulation
char c4 = '\\';             // Backslash
char c5 = '\'';             // Apostrophe
char c6 = '\0';             // Caractère nul (terminateur de chaîne)

// Code octal
char c7 = '\101';           // 'A' en octal (65)

// Code hexadécimal
char c8 = '\x41';           // 'A' en hexa (0x41 = 65)

// Caractère large (Unicode)
wchar_t wc = L'é';          // Nécessite <wchar.h>
```

### Constantes avec `const`

```c
const int TAILLE_MAX = 100;        // Constante entière
const float PI = 3.14159f;         // Constante flottante
const char SEPARATEUR = ';';       // Constante caractère

// const empêche la modification
TAILLE_MAX = 200;  // ERREUR de compilation!

// const avec pointeurs : attention au placement!
const int *p1;        // Pointeur vers un int constant (valeur non modifiable)
int const *p2;        // Équivalent à p1
int *const p3 = &x;   // Pointeur constant vers un int (adresse non modifiable)
const int *const p4 = &x;  // Les deux sont constants
```

### Macros `#define`

```c
#define PI 3.14159
#define MAX_SIZE 1024
#define CARRE(x) ((x) * (x))
#define MESSAGE "Bonjour"

// Utilisation
double aire = PI * r * r;
char buffer[MAX_SIZE];
int result = CARRE(5);     // Devient ((5) * (5))
printf(MESSAGE);

// Différences avec const :
// - #define : remplacement textuel par le préprocesseur
// - const : vraie variable en mémoire avec type
// - #define ne respecte pas les portées
// - const est typée et vérifiée par le compilateur
```

### Constantes d'énumération

```c
// enum crée des constantes entières nommées
enum Couleur {
    ROUGE,      // 0
    VERT,       // 1
    BLEU        // 2
};

enum Mois {
    JANVIER = 1,    // Valeur explicite
    FEVRIER,        // 2 (auto-incrémenté)
    MARS,           // 3
    DECEMBRE = 12
};

// Utilisation
enum Couleur c = ROUGE;
enum Mois m = JANVIER;

// switch pratique avec enum
switch (c) {
    case ROUGE:
        printf("Rouge\n");
        break;
    case VERT:
        printf("Vert\n");
        break;
    case BLEU:
        printf("Bleu\n");
        break;
}
```

---

## Portée et durée de vie des variables

### Portée (scope) : où la variable est visible

```c
#include <stdio.h>

int globale = 100;  // Portée : tout le programme

void fonction(void) {
    int locale_fonction = 20;  // Portée : cette fonction seulement
    printf("globale = %d\n", globale);        // OK
    printf("locale_fonction = %d\n", locale_fonction);  // OK
}

int main(void) {
    int locale_main = 10;  // Portée : main() seulement
    
    printf("globale = %d\n", globale);  // OK
    // printf("locale_fonction = %d\n", locale_fonction);  // ERREUR!
    
    // Portée de bloc
    {
        int locale_bloc = 30;
        printf("locale_bloc = %d\n", locale_bloc);  // OK
    }
    // printf("locale_bloc = %d\n", locale_bloc);  // ERREUR!
    
    // Portée dans une boucle
    for (int i = 0; i < 3; i++) {
        int temp = i * 2;  // Nouvelle variable à chaque itération (conceptuellement)
        printf("%d ", temp);
    }
    // printf("i = %d\n", i);     // ERREUR! i n'existe plus (C99+)
    // printf("temp = %d\n", temp);  // ERREUR!
    
    return 0;
}
```

### Durée de vie (storage duration)

```c
#include <stdio.h>

int globale = 0;  // Durée statique : existe pendant toute l'exécution

void compteur(void) {
    int locale = 0;         // Durée automatique : créée/détruite à chaque appel
    static int persistante = 0;  // Durée statique : conserve sa valeur entre appels
    
    locale++;
    persistante++;
    globale++;
    
    printf("locale=%d, persistante=%d, globale=%d\n", 
           locale, persistante, globale);
}

int main(void) {
    compteur();  // locale=1, persistante=1, globale=1
    compteur();  // locale=1, persistante=2, globale=2
    compteur();  // locale=1, persistante=3, globale=3
    
    return 0;
}
```

### Classes de stockage (storage class specifiers)

| Spécificateur     | Portée  | Durée de vie | Initialisation               |
|-------------------|---------|--------------|------------------------------|
| `auto` (défaut)   | Bloc    | Automatique  | Non initialisée              |
| `static` (local)  | Bloc    | Statique     | Une seule fois, 0 par défaut |
| `static` (global) | Fichier | Statique     | Une seule fois, 0 par défaut |
| `extern`          | Globale | Statique     | Déclaration seulement        |
| `register`        | Bloc    | Automatique  | Suggestion au compilateur    |

```c
// auto : implicite pour variables locales (rarement écrit)
auto int x = 10;  // Équivalent à : int x = 10;

// static local : conserve la valeur entre appels
void fonction(void) {
    static int count = 0;  // Initialisé une seule fois
    count++;
    printf("Appel numéro %d\n", count);
}

// static global : visible uniquement dans ce fichier
static int variable_privee = 42;  // Pas accessible depuis d'autres fichiers

// extern : déclare une variable définie ailleurs
extern int variable_externe;  // Définie dans un autre fichier .c

// register : suggère de stocker en registre CPU (souvent ignoré aujourd'hui)
register int compteur_rapide;
```

### Exemple multi-fichiers avec `extern`

**global.c** :
```c
// Définition de la variable
int compteur_global = 0;
```

**global.h** :
```c
// Déclaration externe
extern int compteur_global;
```

**main.c** :
```c
#include "global.h"

int main(void) {
    compteur_global = 42;  // Utilise la variable définie dans global.c
    return 0;
}
```

---

## Qualificateurs de type

### `const` : valeur immuable

```c
const int MAX = 100;
// MAX = 200;  // ERREUR!

// const avec pointeurs
int x = 10, y = 20;

const int *ptr1 = &x;  // Pointeur vers constante
// *ptr1 = 15;  // ERREUR: ne peut pas modifier la valeur
ptr1 = &y;      // OK: peut changer l'adresse

int *const ptr2 = &x;  // Pointeur constant
*ptr2 = 15;     // OK: peut modifier la valeur
// ptr2 = &y;  // ERREUR: ne peut pas changer l'adresse

const int *const ptr3 = &x;  // Les deux sont constants
// *ptr3 = 15;  // ERREUR
// ptr3 = &y;   // ERREUR
```

### `volatile` : empêche les optimisations

Utilisé pour les variables qui peuvent changer de manière imprévisible :

```c
// Registre matériel (peut changer à tout moment)
volatile unsigned int *status_register = (unsigned int *)0x40001000;

// Variable modifiée par un signal ou une interruption
volatile int signal_recu = 0;

void handler(int sig) {
    signal_recu = 1;  // Modifié de manière asynchrone
}

int main(void) {
    while (!signal_recu) {
        // Sans volatile, le compilateur pourrait optimiser cette boucle
        // en "while(1)" car il ne voit pas de modification de signal_recu
    }
    return 0;
}
```

### `restrict` (C99+) : optimisation des pointeurs

Indique au compilateur que ce pointeur est le seul moyen d'accéder à cette mémoire :

```c
// Le compilateur peut optimiser plus agressivement
void copier(int * restrict dest, const int * restrict src, size_t n) {
    for (size_t i = 0; i < n; i++) {
        dest[i] = src[i];
    }
}

// Sans restrict, le compilateur doit supposer que dest et src pourraient
// pointer vers la même zone mémoire
```

---

## Conversions de types (casting)

### Conversions implicites (automatiques)

```c
int i = 42;
float f = i;          // int → float automatique (promotion)
double d = f;         // float → double automatique

char c = 'A';
int ascii = c;        // char → int automatique

// Hiérarchie de conversion (promotion)
// char → short → int → long → long long
//                 ↓         ↓
// float ←───── double ← long double

// Dans les expressions mixtes, le type le plus "grand" gagne
int a = 5;
double b = 2.5;
double resultat = a + b;  // a converti en double, résultat double

// ATTENTION aux pertes de données
int grand = 1000;
char petit = grand;  // Troncature! petit = -24 (1000 % 256 avec signé)
```

### Conversions explicites (cast)

```c
// Syntaxe : (type) expression
double d = 3.99;
int i = (int)d;  // Troncature : i = 3 (pas d'arrondi!)

// Division entière vs flottante
int a = 5, b = 2;
double r1 = a / b;        // Division entière d'abord : r1 = 2.0
double r2 = (double)a / b;  // a converti d'abord : r2 = 2.5
double r3 = a / (double)b;  // b converti d'abord : r3 = 2.5
double r4 = (double)(a / b);  // Division entière, puis conversion : r4 = 2.0

// Cast de pointeurs (usage avancé/dangereux)
int x = 0x41424344;
char *p = (char *)&x;  // Réinterprète l'int comme tableau de char
// Sur little-endian : p[0]='D', p[1]='C', p[2]='B', p[3]='A'

// void* peut être converti vers n'importe quel type de pointeur
void *generic = malloc(100);
int *arr = (int *)generic;  // Cast nécessaire en C++ mais optionnel en C
```

### Pièges courants

```c
// Piège 1 : Perte de données silencieuse
int big = 100000;
short small = big;  // Troncature sans avertissement!
printf("%d\n", small);  // -31072 (pas 100000!)

// Piège 2 : Conversion signed/unsigned
int negative = -1;
unsigned int positive = negative;
printf("%u\n", positive);  // 4294967295!

if (negative < positive) {  // FAUX! negative converti en unsigned
    printf("Attention!\n");  // Ne s'exécute pas!
}

// Piège 3 : Comparaison signed/unsigned avec size_t
int index = -1;
size_t taille = 10;
if (index < taille) {  // FAUX! index devient très grand
    printf("Dans les bornes\n");  // Ne s'exécute pas!
}

// Solution
if (index >= 0 && (size_t)index < taille) {
    // Sûr
}
```

---

## Entrées/Sorties formatées

### `printf` : sortie formatée

```c
#include <stdio.h>

int main(void) {
    int i = 42;
    float f = 3.14159f;
    double d = 2.71828;
    char c = 'X';
    char s[] = "Hello";
    
    // Spécificateurs de base
    printf("Entier : %d\n", i);          // Décimal signé
    printf("Entier : %i\n", i);          // Équivalent à %d
    printf("Non signé : %u\n", 42u);     // Décimal non signé
    printf("Octal : %o\n", 42);          // Octal : 52
    printf("Hexa min : %x\n", 255);      // Hexa minuscules : ff
    printf("Hexa maj : %X\n", 255);      // Hexa majuscules : FF
    printf("Float : %f\n", f);           // 3.141590
    printf("Double : %lf\n", d);         // 2.718280 (%f suffit aussi)
    printf("Scientifique : %e\n", d);    // 2.718280e+00
    printf("Caractère : %c\n", c);       // X
    printf("Chaîne : %s\n", s);          // Hello
    printf("Pointeur : %p\n", (void*)&i); // 0x7ffd...
    printf("Pourcent : %%\n");           // %
    
    // Formatage avancé : largeur et précision
    printf("[%10d]\n", i);     // [        42] - largeur 10, aligné droite
    printf("[%-10d]\n", i);    // [42        ] - aligné gauche
    printf("[%010d]\n", i);    // [0000000042] - padding avec zéros
    printf("[%+d]\n", i);      // [+42] - affiche toujours le signe
    
    printf("[%.2f]\n", f);     // [3.14] - 2 décimales
    printf("[%10.2f]\n", f);   // [      3.14] - largeur 10, 2 décimales
    printf("[%10.4s]\n", "Hello");  // [      Hell] - max 4 caractères
    
    // Types longs
    long l = 1234567890L;
    long long ll = 9876543210LL;
    printf("long : %ld\n", l);
    printf("long long : %lld\n", ll);
    
    // size_t
    size_t sz = sizeof(int);
    printf("size_t : %zu\n", sz);
    
    return 0;
}
```

### Tableau des spécificateurs

| Spécificateur | Type                        | Exemple              |
|---------------|-----------------------------|----------------------|
| `%d`, `%i`    | int (signé)                 | -42, 42              |
| `%u`          | unsigned int                | 42                   |
| `%o`          | unsigned int (octal)        | 52                   |
| `%x`, `%X`    | unsigned int (hexa)         | 2a, 2A               |
| `%f`          | float/double                | 3.141590             |
| `%e`, `%E`    | float/double (scientifique) | 3.14e+00             |
| `%g`, `%G`    | float/double (compact)      | 3.14159              |
| `%c`          | char                        | A                    |
| `%s`          | char* (chaîne)              | Hello                |
| `%p`          | void* (pointeur)            | 0x7fff...            |
| `%%`          | (littéral)                  | %                    |
| `%ld`         | long                        | -1234567890          |
| `%lld`        | long long                   | -9223372036854775807 |
| `%lu`         | unsigned long               | 1234567890           |
| `%llu`        | unsigned long long          | 18446744073709551615 |
| `%zu`         | size_t                      | 8                    |
| `%zd`         | ssize_t                     | -1                   | 

### `scanf` : entrée formatée

```c
#include <stdio.h>

int main(void) {
    int i;
    float f;
    char c;
    char s[100];
    
    // Lecture basique (ATTENTION: utiliser & pour les variables!)
    printf("Entrez un entier : ");
    scanf("%d", &i);  // &i = adresse de i
    
    printf("Entrez un flottant : ");
    scanf("%f", &f);
    
    printf("Entrez un caractère : ");
    scanf(" %c", &c);  // Espace avant %c pour ignorer whitespace
    
    printf("Entrez un mot : ");
    scanf("%99s", s);  // Pas de & pour les tableaux! Limite à 99 chars
    
    // Lecture de ligne entière (préférer fgets)
    char ligne[200];
    printf("Entrez une ligne : ");
    getchar();  // Consomme le \n restant
    fgets(ligne, sizeof(ligne), stdin);  // Plus sûr que gets()
    
    // Vérification du succès de scanf
    int lu;
    printf("Entrez deux entiers : ");
    lu = scanf("%d %d", &i, &f);  // Retourne le nombre d'éléments lus
    if (lu != 2) {
        printf("Erreur de lecture!\n");
    }
    
    return 0;
}
```

### Sécurité de `scanf`

```c
// DANGEREUX : pas de limite de taille
char buffer[10];
scanf("%s", buffer);  // Buffer overflow si entrée > 9 chars!

// SÉCURISÉ : limite explicite
scanf("%9s", buffer);  // Maximum 9 caractères + '\0'

// ENCORE MIEUX : utiliser fgets pour les chaînes
fgets(buffer, sizeof(buffer), stdin);
// Attention: fgets conserve le '\n' final
buffer[strcspn(buffer, "\n")] = '\0';  // Supprimer le \n
```

---

## Comparaison avec Python/JS/C#/Go

### Python vs C : Typage et déclaration

```python
# Python : typage dynamique
x = 42          # int automatique
x = "hello"     # Réaffectation en str, OK
x = 3.14        # Réaffectation en float, OK

# Type hints (optionnels, non vérifiés à l'exécution)
age: int = 25
nom: str = "Alice"
```

```c
// C : typage statique
int x = 42;
// x = "hello";  // ERREUR!
// x = 3.14;     // Warning, troncature

// Types obligatoires et fixés
int age = 25;
char nom[] = "Alice";
```

### JavaScript vs C : Types primitifs

```javascript
// JavaScript : types dynamiques, coercition automatique
let x = 42;           // number
let y = "42";         // string
console.log(x + y);   // "4242" (concaténation!)
console.log(x == y);  // true (coercition)
console.log(x === y); // false (strict)

// Types: number, string, boolean, null, undefined, symbol, bigint
```

```c
// C : pas de coercition de chaînes
int x = 42;
char y[] = "42";
// int sum = x + y;  // ERREUR! Types incompatibles

// Pour convertir : fonctions explicites
#include <stdlib.h>
int y_int = atoi("42");  // String → int
char buffer[20];
sprintf(buffer, "%d", x);  // int → string
```

### C# vs C : Types valeur et référence

```csharp
// C# : types valeur (stack) et référence (heap)
int x = 42;           // Type valeur, sur la stack
string s = "Hello";   // Type référence, sur la heap

// Nullable types
int? nullable = null;  // Type valeur nullable

// var : inférence de type
var y = 3.14;  // double inféré
```

```c
// C : tout est "valeur" par défaut
int x = 42;           // Sur la stack
char s[] = "Hello";   // Tableau sur la stack

// Pas de null pour les types primitifs
// int nullable = null;  // ERREUR!

// Pour les pointeurs seulement
int *ptr = NULL;  // Pointeur nul

// Pas d'inférence (sauf en C23 avec auto)
// auto y = 3.14;  // C23 seulement
```

### Go vs C : Déclaration et typage

```go
// Go : typage statique avec inférence
var x int = 42      // Déclaration explicite
y := 42             // Inférence avec :=
var z = 42          // Inférence avec var

// Types stricts, pas de conversion implicite
var a int = 10
var b float64 = float64(a)  // Cast explicite obligatoire

// Zero values (initialisation automatique)
var i int       // 0
var f float64   // 0.0
var s string    // ""
var p *int      // nil
```

```c
// C : pas d'inférence (avant C23)
int x = 42;
// y := 42;  // Syntaxe invalide

// Conversions implicites (parfois dangereuses)
int a = 10;
float b = a;  // Conversion implicite OK

// Pas d'initialisation automatique des variables locales!
int i;       // Valeur indéterminée!
float f;     // Valeur indéterminée!
```

### Tableau comparatif complet
 
| Caractéristique       | C                | Python     | JavaScript          | C#              | Go             |
|-----------------------|------------------|------------|---------------------|-----------------|----------------|
| Typage                | Statique, faible | Dynamique  | Dynamique           | Statique        | Statique       |
| Déclaration type      | Obligatoire      | Non        | Non (var/let/const) | Obligatoire/var | Obligatoire/:= |
| Inférence             | C23 (auto)       | Native     | Native              | var             | Native         |
| Null pour primitifs   | Non              | None       | null/undefined      | Nullable<T>     | Zero value     |
| Conversion implicite  | Oui              | Non        | Oui (coercition)    | Limitée         | Non            |
| Vérification overflow | Non              | Oui (auto) | Non                 | Oui (checked)   | Non (wrap)     |
| Taille types fixe     | Non              | N/A        | N/A                 | Oui             | Oui            |

---

## Exercices du chapitre

### Exercice 2.1 : Exploration des types (Débutant)

Créez un programme qui affiche la taille en octets de tous les types fondamentaux :

```c
printf("char : %zu octets\n", sizeof(char));
// Continuer pour short, int, long, long long, float, double, long double
```

**Attendu** : Un tableau complet des tailles sur votre système.

---

### Exercice 2.2 : Limites des types (Débutant)

Utilisez `<limits.h>` et `<float.h>` pour afficher les valeurs minimum et maximum de :
- `signed char`, `unsigned char`
- `int`, `unsigned int`
- `long long`
- `float`, `double`

---

### Exercice 2.3 : Conversions dangereuses (Intermédiaire)

Analysez ce code et prédisez les sorties AVANT de l'exécuter :

```c
#include <stdio.h>

int main(void) {
    unsigned int u = 10;
    int s = -5;
    
    if (s < u) {
        printf("s < u\n");
    } else {
        printf("s >= u\n");
    }
    
    printf("s + u = %u\n", s + u);
    printf("s + u = %d\n", s + u);
    
    return 0;
}
```

Expliquez pourquoi.

---

### Exercice 2.4 : Calculatrice de type (Intermédiaire)

Créez un programme qui :
1. Demande à l'utilisateur un type de calcul (1=add, 2=sub, 3=mul, 4=div)
2. Demande deux nombres flottants
3. Effectue le calcul et affiche le résultat avec 4 décimales
4. Gère la division par zéro

---

### Exercice 2.5 : Conversion de températures (Intermédiaire)

Créez un programme qui :
1. Demande une température et son unité (C, F, K)
2. Convertit et affiche dans les deux autres unités
3. Utilise des constantes pour les formules

Formules :
- C = (F - 32) × 5/9
- K = C + 273.15

---

### Exercice 2.6 : Manipulation de bits dans un octet (Intermédiaire-Avancé)

Utilisez `unsigned char` (1 octet = 8 bits) pour :
1. Afficher un nombre en binaire
2. Compter le nombre de bits à 1
3. Inverser tous les bits
4. Vérifier si un bit spécifique est à 1

```c
unsigned char valeur = 0b10110011;  // 179 en décimal
// Sortie attendue :
// Binaire : 10110011
// Bits à 1 : 5
// Inversé : 01001100 (76)
// Bit 0 : 1
// Bit 2 : 0
// ...
```

---

### Exercice 2.7 : Variable statique compteur (Intermédiaire)

Créez une fonction `int generer_id(void)` qui :
- Retourne un identifiant unique à chaque appel (1, 2, 3, ...)
- Utilise une variable `static` pour conserver le compteur
- N'utilise aucune variable globale

---

### Exercice 2.8 : Précision flottante (Avancé)

Écrivez un programme qui démontre les problèmes de précision des flottants :
1. Calculez 0.1 + 0.2 et comparez à 0.3
2. Implémentez une fonction de comparaison avec epsilon
3. Montrez l'accumulation d'erreurs en additionnant 0.1 dix mille fois

---

### Exercice 2.9 : Parser d'arguments (Avancé)

Créez un programme qui accepte des arguments de ligne de commande :

```bash
./programme -n 42 -f 3.14 -s "hello" -v
```

Et affiche :
```
Entier n : 42
Flottant f : 3.140000
Chaîne s : hello
Verbose : activé
```

Utilisez `argc`, `argv`, `atoi()`, `atof()`, et `strcmp()`.

---

## Résumé du chapitre

✅ **Types fondamentaux** : `char`, `short`, `int`, `long`, `long long`, `float`, `double`

✅ **Modificateurs** : `signed`, `unsigned` changent la plage de valeurs

✅ **Types fixes (C99+)** : `int32_t`, `uint64_t`, `size_t` pour la portabilité

✅ **Constantes** : `const`, `#define`, `enum` selon les besoins

✅ **Portée** : locale (bloc), globale (fichier/programme)

✅ **Durée de vie** : automatique (stack), statique (`static`/globale)

✅ **Conversions** : implicites (dangereuses parfois) et explicites (cast)

✅ **Entrées/Sorties** : `printf`/`scanf` avec spécificateurs de format

✅ **Différence majeure vs Python/JS** : typage statique, pas de conversion automatique de chaînes

---

**Prochain chapitre** : Opérateurs (arithmétiques, logiques, bit-à-bit)
