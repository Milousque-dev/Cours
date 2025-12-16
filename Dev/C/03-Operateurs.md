# Chapitre 3 : Opérateurs

## Table des matières

1. [Introduction aux opérateurs](#introduction-aux-opérateurs)
2. [Opérateurs arithmétiques](#opérateurs-arithmétiques)
3. [Opérateurs de comparaison (relationnels)](#opérateurs-de-comparaison-relationnels)
4. [Opérateurs logiques](#opérateurs-logiques)
5. [Opérateurs bit-à-bit (bitwise)](#opérateurs-bit-à-bit-bitwise)
6. [Opérateurs d'affectation](#opérateurs-daffectation)
7. [Opérateurs d'incrémentation et décrémentation](#opérateurs-dincrémentation-et-décrémentation)
8. [Opérateur ternaire (conditionnel)](#opérateur-ternaire-conditionnel)
9. [Opérateurs spéciaux](#opérateurs-spéciaux)
10. [Précédence et associativité](#précédence-et-associativité)
11. [Comparaison avec Python/JS/C#/Go](#comparaison-avec-pythonjscgo)
12. [Exercices du chapitre](#exercices-du-chapitre)

---

## Introduction aux opérateurs

### Définition

Un **opérateur** est un symbole qui indique au compilateur d'effectuer une opération sur une ou plusieurs **opérandes**. Le C possède un ensemble riche d'opérateurs, notamment des opérateurs bit-à-bit rarement disponibles dans les langages de haut niveau.

### Classification des opérateurs

```
Opérateurs C
├── Par nombre d'opérandes
│   ├── Unaires    (1 opérande) : ++, --, !, ~, &, *, sizeof
│   ├── Binaires   (2 opérandes) : +, -, *, /, %, ==, &&, |
│   └── Ternaire   (3 opérandes) : ? :
│
├── Par fonction
│   ├── Arithmétiques : +, -, *, /, %
│   ├── Relationnels  : ==, !=, <, >, <=, >=
│   ├── Logiques      : &&, ||, !
│   ├── Bit-à-bit     : &, |, ^, ~, <<, >>
│   ├── Affectation   : =, +=, -=, *=, /=, %=, &=, |=, ^=, <<=, >>=
│   ├── Incrémentation: ++, --
│   └── Spéciaux      : sizeof, &, *, ->, ., (type), ?:, ,
```

---

## Opérateurs arithmétiques

### Opérateurs de base

| Opérateur | Nom            | Exemple | Résultat       |
|-----------|----------------|---------|----------------|
| `+`       | Addition       | `5 + 3` | `8`            |
| `-`       | Soustraction   | `5 - 3` | `2`            |
| `*`       | Multiplication | `5 * 3` | `15`           |
| `/`       | Division       | `5 / 3` | `1` (entière!) |
| `%`       | Modulo (reste) | `5 % 3` | `2`            |       
| `-`       | Moins unaire   | `-5`    | `-5`           |
| `+`       | Plus unaire    | `+5`    | `5`            |

### Division entière vs flottante

```c
#include <stdio.h>

int main(void) {
    // Division entre deux entiers = division entière (troncature)
    int a = 7, b = 3;
    int resultat_int = a / b;  // 2 (pas 2.333...)
    printf("7 / 3 (int) = %d\n", resultat_int);
    
    // Pour obtenir une division flottante, au moins un opérande doit être flottant
    double resultat_double1 = (double)a / b;     // Cast explicite
    double resultat_double2 = a / (double)b;     // Autre méthode
    double resultat_double3 = 7.0 / 3;           // Littéral flottant
    printf("7 / 3 (double) = %f\n", resultat_double1);  // 2.333333
    
    // Piège courant!
    double piege = 7 / 3;  // Division entière d'abord, puis conversion
    printf("Piège : %f\n", piege);  // 2.000000 (pas 2.333333!)
    
    // Division par zéro
    // int div_zero = 5 / 0;  // Comportement indéfini! (crash probable)
    double inf = 5.0 / 0.0;   // Résultat : inf (infini)
    double nan_val = 0.0 / 0.0;  // Résultat : nan (Not a Number)
    
    return 0;
}
```

### Opérateur modulo (%)

L'opérateur `%` retourne le **reste** de la division entière :

```c
#include <stdio.h>

int main(void) {
    // Cas de base
    printf("10 %% 3 = %d\n", 10 % 3);   // 1
    printf("10 %% 5 = %d\n", 10 % 5);   // 0 (divisible)
    printf("3 %% 10 = %d\n", 3 % 10);   // 3
    
    // Avec nombres négatifs : le signe du résultat suit le dividende (C99+)
    printf("-10 %% 3 = %d\n", -10 % 3);   // -1
    printf("10 %% -3 = %d\n", 10 % -3);   // 1
    printf("-10 %% -3 = %d\n", -10 % -3); // -1
    
    // Applications pratiques
    int nombre = 1234;
    int dernier_chiffre = nombre % 10;  // 4
    
    // Vérifier si pair ou impair
    if (nombre % 2 == 0) {
        printf("%d est pair\n", nombre);
    } else {
        printf("%d est impair\n", nombre);
    }
    
    // Rebouclage (wrap-around)
    for (int i = 0; i < 15; i++) {
        printf("%d ", i % 5);  // 0 1 2 3 4 0 1 2 3 4 0 1 2 3 4
    }
    
    // Note : % ne fonctionne qu'avec les entiers
    // double reste = 5.5 % 2.0;  // ERREUR!
    // Utiliser fmod() de <math.h> pour les flottants
    
    return 0;
}
```

### Débordements arithmétiques

```c
#include <stdio.h>
#include <limits.h>

int main(void) {
    // Débordement d'entier signé : COMPORTEMENT INDÉFINI!
    int max = INT_MAX;  // 2147483647
    printf("INT_MAX = %d\n", max);
    printf("INT_MAX + 1 = %d\n", max + 1);  // Indéfini! (souvent -2147483648)
    
    // Débordement d'entier non signé : wrapping défini (modulo 2^n)
    unsigned int umax = UINT_MAX;  // 4294967295
    printf("UINT_MAX = %u\n", umax);
    printf("UINT_MAX + 1 = %u\n", umax + 1);  // Toujours 0
    
    // Multiplication avec débordement
    int a = 50000;
    int b = 50000;
    int produit = a * b;  // Débordement! 2500000000 > INT_MAX
    printf("50000 * 50000 = %d\n", produit);  // Résultat incorrect
    
    // Solution : utiliser un type plus grand
    long long produit_safe = (long long)a * b;
    printf("50000 * 50000 (safe) = %lld\n", produit_safe);  // 2500000000
    
    return 0;
}
```

---

## Opérateurs de comparaison (relationnels)

Les opérateurs relationnels comparent deux valeurs et retournent un résultat **booléen** (0 pour faux, 1 pour vrai) :

| Opérateur | Signification     | Exemple  | Résultat   |
|-----------|-------------------|----------|------------|
| `==`      | Égal à            | `5 == 5` | `1` (vrai) |
| `!=`      | Différent de      | `5 != 3` | `1` (vrai) |
| `<`       | Inférieur à       | `3 < 5`  | `1` (vrai) |
| `>`       | Supérieur à       | `3 > 5`  | `0` (faux) |
| `<=`      | Inférieur ou égal | `5 <= 5` | `1` (vrai) |
| `>=`      | Supérieur ou égal | `3 >= 5` | `0` (faux) |

### Exemples et pièges

```c
#include <stdio.h>
#include <stdbool.h>

int main(void) {
    int a = 5, b = 3, c = 5;
    
    // Comparaisons de base
    printf("5 == 5 : %d\n", a == c);   // 1
    printf("5 != 3 : %d\n", a != b);   // 1
    printf("5 > 3 : %d\n", a > b);     // 1
    printf("5 < 3 : %d\n", a < b);     // 0
    
    // PIÈGE 1 : Confusion = et ==
    int x = 10;
    if (x = 5) {  // AFFECTATION, pas comparaison! x devient 5
        printf("Vrai!\n");  // S'exécute car 5 est non-nul
    }
    // Compilateur: "warning: suggest parentheses around assignment"
    
    // Bonne pratique : mettre la constante à gauche
    if (5 == x) {  // "5 = x" serait une erreur de compilation
        printf("x vaut 5\n");
    }
    
    // PIÈGE 2 : Comparaison de flottants
    float f1 = 0.1f + 0.2f;
    float f2 = 0.3f;
    if (f1 == f2) {
        printf("Égaux\n");  // Probablement PAS exécuté!
    } else {
        printf("Différents : %.10f vs %.10f\n", f1, f2);
    }
    
    // Solution : comparaison avec epsilon
    float epsilon = 1e-6f;
    if (fabsf(f1 - f2) < epsilon) {
        printf("Approximativement égaux\n");
    }
    
    // PIÈGE 3 : Comparaison chaînée (NE FONCTIONNE PAS comme en maths!)
    int val = 5;
    // if (1 < val < 10)  // Évalué comme : (1 < val) < 10, soit 1 < 10 = vrai!
    // Ce n'est PAS "val entre 1 et 10"
    
    // Bonne façon :
    if (val > 1 && val < 10) {
        printf("%d est entre 1 et 10\n", val);
    }
    
    // PIÈGE 4 : Comparaison signed/unsigned
    int negatif = -1;
    unsigned int positif = 1;
    if (negatif < positif) {
        printf("-1 < 1\n");  // NON EXÉCUTÉ!
    } else {
        printf("-1 >= 1 ???\n");  // Exécuté car -1 converti en UINT_MAX
    }
    
    return 0;
}
```

---

## Opérateurs logiques

Les opérateurs logiques permettent de combiner des conditions booléennes :

| Opérateur | Nom               | Exemple    | Résultat |
|-----------|-------------------|------------|----------|
| `&&`      | ET logique (AND)  | `1 && 0`   | `0`      |
| `\|\|`    | OU logique (OR)   | `1 \|\| 0` | `1`      |
| `!`       | NON logique (NOT) | `!1`       | `0`      |

### Tables de vérité

```
AND (&&)           OR (||)           NOT (!)
A   B   A&&B       A   B   A||B      A    !A
0   0    0         0   0    0        0     1
0   1    0         0   1    1        1     0
1   0    0         1   0    1
1   1    1         1   1    1
```

### Évaluation en court-circuit (short-circuit)

C garantit l'évaluation **de gauche à droite** avec **court-circuit** :

```c
#include <stdio.h>

int fonction_test(void) {
    printf("fonction_test() appelée\n");
    return 1;
}

int main(void) {
    int a = 0, b = 1;
    
    // Court-circuit AND : si le premier est faux, le second n'est pas évalué
    if (a && fonction_test()) {
        printf("Les deux vrais\n");
    }
    // "fonction_test() appelée" N'est PAS affiché car a est faux
    
    // Court-circuit OR : si le premier est vrai, le second n'est pas évalué
    if (b || fonction_test()) {
        printf("Au moins un vrai\n");
    }
    // "fonction_test() appelée" N'est PAS affiché car b est vrai
    
    // Application pratique : éviter les erreurs
    int *ptr = NULL;
    // Sécuritaire grâce au court-circuit :
    if (ptr != NULL && *ptr > 0) {
        printf("Valeur positive\n");
    }
    // Si ptr est NULL, *ptr n'est pas évalué (crash évité!)
    
    // Vérification de bornes avant accès tableau
    int index = 100;
    int tableau[10];
    if (index >= 0 && index < 10 && tableau[index] > 0) {
        // Accès sûr
    }
    
    return 0;
}
```

### Valeurs de vérité en C

En C, **toute valeur non nulle est vraie**, et **zéro est faux** :

```c
#include <stdio.h>

int main(void) {
    // Ces conditions sont TOUTES vraies
    if (42) printf("42 est vrai\n");
    if (-1) printf("-1 est vrai\n");
    if (3.14) printf("3.14 est vrai\n");
    if ("hello") printf("\"hello\" est vrai (pointeur non nul)\n");
    
    // Ces conditions sont TOUTES fausses
    int zero = 0;
    float zero_f = 0.0;
    char *null_ptr = NULL;
    
    if (!zero) printf("0 est faux\n");
    if (!zero_f) printf("0.0 est faux\n");
    if (!null_ptr) printf("NULL est faux\n");
    
    // L'opérateur ! convertit en 0 ou 1
    printf("!42 = %d\n", !42);     // 0
    printf("!!42 = %d\n", !!42);   // 1 (double négation = normalisation)
    printf("!0 = %d\n", !0);       // 1
    
    return 0;
}
```

---

## Opérateurs bit-à-bit (bitwise)

Les opérateurs bit-à-bit travaillent sur les **représentations binaires** des entiers. Ils sont essentiels pour :
- La programmation système et embarquée
- Les protocoles réseau
- La cryptographie
- L'optimisation de performances

### Vue d'ensemble

| Opérateur | Nom             | Exemple (8 bits)            | Résultat     |
|-----------|-----------------|-----------------------------|--------------|
| `&`       | AND bit-à-bit   | `0b11001100 & 0b10101010`   | `0b10001000` |
| `\|`      | OR bit-à-bit    | `0b11001100 \| 0b10101010`  | `0b11101110` |
| `^`       | XOR bit-à-bit   | `0b11001100 ^ 0b10101010`   | `0b01100110` |
| `~`       | NOT bit-à-bit   | `~0b11001100`               | `0b00110011` |
| `<<`      | Décalage gauche | `0b00001111 << 2`           | `0b00111100` |
| `>>`      | Décalage droite | `0b11110000 >> 2`           | `0b00111100` |

### AND bit-à-bit (&)

Chaque bit du résultat est 1 **seulement si** les deux bits correspondants sont 1 :

```c
#include <stdio.h>

void afficher_binaire(unsigned char n) {
    for (int i = 7; i >= 0; i--) {
        printf("%d", (n >> i) & 1);
    }
    printf("\n");
}

int main(void) {
    unsigned char a = 0b11001100;  // 204
    unsigned char b = 0b10101010;  // 170
    unsigned char resultat = a & b;
    
    printf("  ");
    afficher_binaire(a);
    printf("& ");
    afficher_binaire(b);
    printf("= ");
    afficher_binaire(resultat);  // 10001000 (136)
    
    // Applications pratiques :
    
    // 1. Masquer des bits (extraire certains bits)
    unsigned char valeur = 0b11010110;
    unsigned char masque_bas = 0x0F;  // 00001111
    unsigned char bits_bas = valeur & masque_bas;  // 00000110
    
    // 2. Tester si un bit est à 1
    unsigned char flags = 0b10110100;
    if (flags & (1 << 4)) {  // Teste le bit 4
        printf("Bit 4 est à 1\n");
    }
    
    // 3. Forcer un bit à 0
    flags = flags & ~(1 << 2);  // Met le bit 2 à 0
    
    // 4. Vérifier si un nombre est pair (bit 0 = 0)
    int nombre = 42;
    if ((nombre & 1) == 0) {
        printf("%d est pair\n", nombre);
    }
    
    return 0;
}
```

### OR bit-à-bit (|)

Chaque bit du résultat est 1 si **au moins un** des bits correspondants est 1 :

```c
#include <stdio.h>

int main(void) {
    unsigned char a = 0b11001100;
    unsigned char b = 0b10101010;
    unsigned char resultat = a | b;  // 11101110 (238)
    
    // Applications pratiques :
    
    // 1. Mettre un bit à 1
    unsigned char flags = 0b00000000;
    flags = flags | (1 << 3);  // Met le bit 3 à 1 : 00001000
    flags |= (1 << 5);         // Met le bit 5 à 1 : 00101000
    
    // 2. Combiner des drapeaux
    #define FLAG_READ    0x01  // 00000001
    #define FLAG_WRITE   0x02  // 00000010
    #define FLAG_EXECUTE 0x04  // 00000100
    
    unsigned char permissions = FLAG_READ | FLAG_WRITE;  // 00000011
    
    // 3. Forcer des bits à 1
    unsigned char valeur = 0b10100000;
    valeur |= 0x0F;  // Force les 4 bits bas à 1 : 10101111
    
    return 0;
}
```

### XOR bit-à-bit (^)

Chaque bit du résultat est 1 si les bits correspondants sont **différents** :

```c
#include <stdio.h>

int main(void) {
    unsigned char a = 0b11001100;
    unsigned char b = 0b10101010;
    unsigned char resultat = a ^ b;  // 01100110 (102)
    
    // Propriétés importantes :
    // a ^ a = 0 (un nombre XOR lui-même = 0)
    // a ^ 0 = a (XOR avec 0 ne change rien)
    // a ^ b ^ b = a (double XOR = annulation)
    
    // Applications pratiques :
    
    // 1. Inverser des bits spécifiques (toggle)
    unsigned char flags = 0b11110000;
    flags ^= (1 << 4);  // Inverse le bit 4 : 11100000
    flags ^= (1 << 4);  // Inverse encore : 11110000
    
    // 2. Échanger deux variables SANS variable temporaire
    int x = 5, y = 10;
    x = x ^ y;   // x = 5 ^ 10 = 15
    y = x ^ y;   // y = 15 ^ 10 = 5
    x = x ^ y;   // x = 15 ^ 5 = 10
    printf("x = %d, y = %d\n", x, y);  // x = 10, y = 5
    
    // 3. Cryptographie simple (chiffrement XOR)
    char message[] = "Hello";
    char cle = 0x5A;
    
    // Chiffrement
    for (int i = 0; message[i] != '\0'; i++) {
        message[i] ^= cle;
    }
    printf("Chiffré : %s\n", message);
    
    // Déchiffrement (même opération!)
    for (int i = 0; message[i] != '\0'; i++) {
        message[i] ^= cle;
    }
    printf("Déchiffré : %s\n", message);
    
    // 4. Détecter si deux nombres sont de signes différents
    int m = -5, n = 10;
    if ((m ^ n) < 0) {
        printf("Signes différents\n");
    }
    
    return 0;
}
```

### NOT bit-à-bit (~)

Inverse **tous les bits** (complément à 1) :

```c
#include <stdio.h>

int main(void) {
    unsigned char a = 0b00001111;  // 15
    unsigned char resultat = ~a;   // 11110000 (240)
    
    // Attention avec les entiers signés!
    int x = 0;
    int not_x = ~x;  // -1 (tous les bits à 1 en complément à 2)
    printf("~0 = %d\n", not_x);
    
    // Application : créer des masques
    unsigned char masque_bit_3 = (1 << 3);    // 00001000
    unsigned char inverse = ~masque_bit_3;    // 11110111
    
    // Mettre un bit à 0
    unsigned char flags = 0b11111111;
    flags = flags & ~(1 << 5);  // Met bit 5 à 0 : 11011111
    
    return 0;
}
```

### Décalages (<<, >>)

#### Décalage à gauche (<<)

Multiplie par 2^n (décale les bits vers la gauche, remplit avec des 0) :

```c
#include <stdio.h>

int main(void) {
    unsigned char a = 0b00000101;  // 5
    
    printf("%d << 1 = %d\n", a, a << 1);  // 10 (00001010)
    printf("%d << 2 = %d\n", a, a << 2);  // 20 (00010100)
    printf("%d << 3 = %d\n", a, a << 3);  // 40 (00101000)
    
    // Équivalent mathématique : a * 2^n
    int x = 5;
    printf("%d * 8 = %d\n", x, x << 3);  // 5 * 2³ = 40
    
    // Créer des masques de bits
    unsigned int bit_0 = 1 << 0;   // 00000001
    unsigned int bit_4 = 1 << 4;   // 00010000
    unsigned int bit_7 = 1 << 7;   // 10000000
    
    // Attention au débordement!
    unsigned char b = 0b10000000;  // 128
    unsigned char shifted = b << 1;  // 0 (le bit sort!)
    
    return 0;
}
```

#### Décalage à droite (>>)

Divise par 2^n (décale les bits vers la droite) :

```c
#include <stdio.h>

int main(void) {
    unsigned char a = 0b10100000;  // 160
    
    printf("%d >> 1 = %d\n", a, a >> 1);  // 80  (01010000)
    printf("%d >> 2 = %d\n", a, a >> 2);  // 40  (00101000)
    printf("%d >> 4 = %d\n", a, a >> 4);  // 10  (00001010)
    
    // Division entière rapide par puissance de 2
    int x = 100;
    printf("%d / 4 = %d\n", x, x >> 2);   // 25
    printf("%d / 8 = %d\n", x, x >> 3);   // 12
    
    // ATTENTION : comportement différent selon signed/unsigned
    signed char s = -128;    // 10000000 en binaire
    unsigned char u = 128;   // 10000000 en binaire
    
    // Décalage arithmétique (signé) : conserve le signe
    printf("signed -128 >> 2 = %d\n", s >> 2);    // -32 (extension du signe)
    
    // Décalage logique (non signé) : remplit avec des 0
    printf("unsigned 128 >> 2 = %d\n", u >> 2);   // 32
    
    return 0;
}
```

### Manipulation de bits : recettes pratiques

```c
#include <stdio.h>

// Macros utilitaires pour manipulation de bits
#define SET_BIT(x, n)     ((x) |= (1U << (n)))      // Met le bit n à 1
#define CLEAR_BIT(x, n)   ((x) &= ~(1U << (n)))     // Met le bit n à 0
#define TOGGLE_BIT(x, n)  ((x) ^= (1U << (n)))      // Inverse le bit n
#define CHECK_BIT(x, n)   (((x) >> (n)) & 1U)       // Retourne la valeur du bit n
#define GET_BITS(x, p, n) (((x) >> (p)) & ((1U << (n)) - 1))  // Extrait n bits à partir de p

int main(void) {
    unsigned int flags = 0;
    
    // Mettre des bits
    SET_BIT(flags, 0);    // flags = 0b00000001
    SET_BIT(flags, 3);    // flags = 0b00001001
    SET_BIT(flags, 7);    // flags = 0b10001001
    printf("Après SET: %u (0x%02X)\n", flags, flags);
    
    // Effacer un bit
    CLEAR_BIT(flags, 3);  // flags = 0b10000001
    printf("Après CLEAR: %u (0x%02X)\n", flags, flags);
    
    // Inverser un bit
    TOGGLE_BIT(flags, 0); // flags = 0b10000000
    TOGGLE_BIT(flags, 0); // flags = 0b10000001
    printf("Après TOGGLE x2: %u (0x%02X)\n", flags, flags);
    
    // Vérifier un bit
    if (CHECK_BIT(flags, 7)) {
        printf("Bit 7 est à 1\n");
    }
    
    // Extraire des bits
    unsigned int valeur = 0b11010110;
    unsigned int bits_2_3 = GET_BITS(valeur, 2, 2);  // Extrait bits 2-3 : 01
    printf("Bits 2-3 de %u : %u\n", valeur, bits_2_3);
    
    // Compter les bits à 1 (population count)
    unsigned int n = 0b10110101;
    int count = 0;
    while (n) {
        count += n & 1;
        n >>= 1;
    }
    printf("Nombre de bits à 1 : %d\n", count);  // 5
    
    // Alternative avec __builtin_popcount (GCC/Clang)
    printf("popcount: %d\n", __builtin_popcount(0b10110101));
    
    return 0;
}
```

---

## Opérateurs d'affectation

### Affectation simple (=)

```c
int x = 10;        // Initialisation
x = 20;            // Affectation

// L'affectation retourne la valeur affectée
int a, b, c;
a = b = c = 0;     // Affectations en chaîne (droite à gauche)

// Utilisation dans une expression
int y;
if ((y = calcul()) > 0) {  // Affecte ET teste
    printf("Positif : %d\n", y);
}
```

### Affectations composées

| Opérateur | Équivalent   | Exemple       |
|-----------|--------------|---------------|
| `+=`      | `x = x + y`  | `x += 5;`     |
| `-=`      | `x = x - y`  | `x -= 5;`     |
| `*=`      | `x = x * y`  | `x *= 2;`     |
| `/=`      | `x = x / y`  | `x /= 2;`     |
| `%=`      | `x = x % y`  | `x %= 3;`     |
| `&=`      | `x = x & y`  | `x &= 0xFF;`  |
| `\|=`     | `x = x \| y` | `x \|= 0x01;` |
| `^=`      | `x = x ^ y`  | `x ^= mask;`  |
| `<<=`     | `x = x << y` | `x <<= 2;`    |
| `>>=`     | `x = x >> y` | `x >>= 1;`    |

```c
#include <stdio.h>

int main(void) {
    int x = 10;
    
    x += 5;    // x = 15
    x -= 3;    // x = 12
    x *= 2;    // x = 24
    x /= 4;    // x = 6
    x %= 4;    // x = 2
    
    // Plus efficace que l'écriture complète
    // x = x + 1;  vs  x += 1;  (ou x++)
    
    // Utile pour les flags
    unsigned int flags = 0;
    flags |= (1 << 3);   // Active flag 3
    flags &= ~(1 << 3);  // Désactive flag 3
    flags ^= (1 << 3);   // Bascule flag 3
    
    return 0;
}
```

---

## Opérateurs d'incrémentation et décrémentation

### Préfixe vs Postfixe

| Opérateur | Nom                 | Action                                     |
|-----------|---------------------|--------------------------------------------|
| `++x`     | Pré-incrémentation  | Incrémente d'abord, puis utilise la valeur |
| `x++`     | Post-incrémentation | Utilise la valeur, puis incrémente         |
| `--x`     | Pré-décrémentation  | Décrémente d'abord, puis utilise la valeur |
| `x--`     | Post-décrémentation | Utilise la valeur, puis décrémente         |

```c
#include <stdio.h>

int main(void) {
    int x = 5;
    
    // Post-incrémentation : utilise PUIS incrémente
    printf("x++ = %d\n", x++);  // Affiche 5
    printf("x = %d\n", x);      // Affiche 6
    
    x = 5;
    
    // Pré-incrémentation : incrémente PUIS utilise
    printf("++x = %d\n", ++x);  // Affiche 6
    printf("x = %d\n", x);      // Affiche 6
    
    // Dans une boucle for, peu importe (généralement)
    for (int i = 0; i < 5; i++) {  // i++ ou ++i : même résultat
        printf("%d ", i);
    }
    printf("\n");
    
    // ATTENTION : comportement indéfini!
    int a = 5;
    // int b = a++ + a++;  // INDÉFINI! Ne jamais écrire ça!
    // int c = ++a + ++a;  // INDÉFINI!
    // int d = a + a++;    // INDÉFINI!
    
    // Règle : ne modifier une variable qu'UNE SEULE fois par expression
    
    return 0;
}
```

### Applications pratiques

```c
#include <stdio.h>

int main(void) {
    // Parcours de tableau avec pointeur
    int arr[] = {10, 20, 30, 40};
    int *ptr = arr;
    
    printf("%d\n", *ptr++);   // Affiche 10, puis ptr pointe sur arr[1]
    printf("%d\n", *++ptr);   // ptr pointe sur arr[2], affiche 30
    
    // Copie de chaîne
    char src[] = "Hello";
    char dest[10];
    char *s = src, *d = dest;
    
    while ((*d++ = *s++) != '\0');  // Copie caractère par caractère
    printf("Copie : %s\n", dest);
    
    // Compteur dans une condition
    int count = 3;
    while (count--) {  // Teste count, puis décrémente
        printf("Compte à rebours : %d\n", count + 1);
    }
    
    return 0;
}
```

---

## Opérateur ternaire (conditionnel)

L'opérateur ternaire `? :` est le seul opérateur à trois opérandes en C :

```c
condition ? valeur_si_vrai : valeur_si_faux
```

### Exemples

```c
#include <stdio.h>

int main(void) {
    int a = 10, b = 20;
    
    // Équivalent à if-else mais en expression
    int max = (a > b) ? a : b;
    printf("Max : %d\n", max);  // 20
    
    // Peut être utilisé dans printf
    int age = 17;
    printf("Vous êtes %s\n", (age >= 18) ? "majeur" : "mineur");
    
    // Assignation conditionnelle
    int x = -5;
    int abs_x = (x >= 0) ? x : -x;  // Valeur absolue
    
    // Ternaires imbriqués (à éviter pour la lisibilité!)
    int note = 75;
    char *grade = (note >= 90) ? "A" :
                  (note >= 80) ? "B" :
                  (note >= 70) ? "C" :
                  (note >= 60) ? "D" : "F";
    printf("Grade : %s\n", grade);  // C
    
    // Utilisation dans une macro
    #define MAX(a, b) ((a) > (b) ? (a) : (b))
    #define MIN(a, b) ((a) < (b) ? (a) : (b))
    #define ABS(x) ((x) >= 0 ? (x) : -(x))
    
    printf("MAX(3, 7) = %d\n", MAX(3, 7));
    printf("MIN(3, 7) = %d\n", MIN(3, 7));
    printf("ABS(-5) = %d\n", ABS(-5));
    
    return 0;
}
```

### Comparaison avec d'autres langages

```c
// C : opérateur ternaire
int max = (a > b) ? a : b;
```

```python
# Python : expression conditionnelle
max = a if a > b else b
```

```javascript
// JavaScript : identique au C
let max = (a > b) ? a : b;

// Ou avec nullish coalescing (différent!)
let val = x ?? defaultValue;  // Si x est null/undefined
```

---

## Opérateurs spéciaux

### sizeof : taille en octets

```c
#include <stdio.h>

int main(void) {
    // sizeof avec types
    printf("sizeof(char) = %zu\n", sizeof(char));       // 1
    printf("sizeof(int) = %zu\n", sizeof(int));         // 4 (typiquement)
    printf("sizeof(double) = %zu\n", sizeof(double));   // 8
    
    // sizeof avec variables
    int x = 42;
    printf("sizeof(x) = %zu\n", sizeof(x));             // 4
    
    // sizeof avec tableaux
    int arr[10];
    printf("sizeof(arr) = %zu\n", sizeof(arr));         // 40 (10 * 4)
    printf("Nombre d'éléments = %zu\n", sizeof(arr) / sizeof(arr[0]));  // 10
    
    // ATTENTION : sizeof avec pointeurs
    void afficher_taille(int arr[]) {
        printf("sizeof dans fonction = %zu\n", sizeof(arr));  // Taille du pointeur!
    }
    
    // sizeof est évalué à la compilation (pas d'effet de bord)
    int n = 5;
    printf("sizeof(n++) = %zu, n = %d\n", sizeof(n++), n);  // n reste 5!
    
    return 0;
}
```

### Opérateur virgule (,)

L'opérateur virgule évalue les expressions de gauche à droite et retourne la valeur de la dernière :

```c
#include <stdio.h>

int main(void) {
    int x, y;
    
    // Évalue 5, puis 10, retourne 10
    x = (5, 10);
    printf("x = %d\n", x);  // 10
    
    // Utile dans les boucles for
    for (int i = 0, j = 10; i < j; i++, j--) {
        printf("i = %d, j = %d\n", i, j);
    }
    
    // Dans une condition while (rare)
    int n = 3;
    while (printf("n = %d\n", n), n--) {
        // ...
    }
    
    return 0;
}
```

### Opérateurs de pointeurs (&, *, ->)

```c
#include <stdio.h>

struct Point {
    int x;
    int y;
};

int main(void) {
    int x = 42;
    
    // & : opérateur d'adresse (référence)
    int *ptr = &x;     // ptr contient l'adresse de x
    printf("Adresse de x : %p\n", (void*)&x);
    
    // * : opérateur de déréférencement
    printf("Valeur pointée : %d\n", *ptr);  // 42
    *ptr = 100;  // Modifie x via le pointeur
    printf("x après modification : %d\n", x);  // 100
    
    // -> : accès membre via pointeur
    struct Point p = {10, 20};
    struct Point *pp = &p;
    
    printf("Accès direct : (%d, %d)\n", p.x, p.y);
    printf("Accès pointeur : (%d, %d)\n", pp->x, pp->y);
    // pp->x est équivalent à (*pp).x
    
    return 0;
}
```

### Opérateur de cast (type)

```c
int x = 10;
double d = (double)x;  // Cast explicite int → double

void *generic = malloc(100);
int *arr = (int *)generic;  // Cast pointeur void* → int*
```

---

## Précédence et associativité

### Tableau de précédence (de la plus haute à la plus basse)
 
| Précédence | Opérateurs                                                              | Associativité   |
|------------|-------------------------------------------------------------------------|-----------------|
| 1 (max)    | `()` `[]` `->` `.` `++` `--` (postfixe)                                 | Gauche → Droite |
| 2          | `++` `--` (préfixe) `+` `-` (unaires) `!` `~` `*` `&` `sizeof` `(type)` | Droite → Gauche |
| 3          | `*` `/` `%`                                                             | Gauche → Droite |
| 4          | `+` `-`                                                                 | Gauche → Droite |
| 5          | `<<` `>>`                                                               | Gauche → Droite |
| 6          | `<` `<=` `>` `>=`                                                       | Gauche → Droite |
| 7          | `==` `!=`                                                               | Gauche → Droite |
| 8          | `&` (bit AND)                                                           | Gauche → Droite |
| 9          | `^` (bit XOR)                                                           | Gauche → Droite |
| 10         | `\|` (bit OR)                                                           | Gauche → Droite |
| 11         | `&&` (AND logique)                                                      | Gauche → Droite |
| 12         | `\|\|` (OR logique)                                                     | Gauche → Droite |
| 13         | `?:` (ternaire)                                                         | Droite → Gauche |
| 14         | `=` `+=` `-=` `*=` `/=` etc.                                            | Droite → Gauche |
| 15 (min)   | `,` (virgule)                                                           | Gauche → Droite |

### Exemples d'évaluation

```c
#include <stdio.h>

int main(void) {
    // Multiplication avant addition
    int a = 2 + 3 * 4;  // 2 + 12 = 14 (pas 20)
    
    // Comparaison avant AND logique
    int b = 5 > 3 && 2 < 4;  // (5 > 3) && (2 < 4) = 1 && 1 = 1
    
    // PIÈGE : bit AND vs comparaison
    int c = 5 & 3 == 3;  // 5 & (3 == 3) = 5 & 1 = 1 (pas ce qu'on veut!)
    int d = (5 & 3) == 3;  // (5 & 3) == 3 = 1 == 3 = 0
    
    // Associativité droite de l'affectation
    int x, y, z;
    x = y = z = 10;  // z=10, puis y=10, puis x=10
    
    // Associativité gauche de la soustraction
    int e = 10 - 5 - 2;  // (10 - 5) - 2 = 3 (pas 10 - 3 = 7)
    
    // CONSEIL : Utilisez des parenthèses pour la clarté!
    int resultat = ((a + b) * c) - (d / e);  // Intent clair
    
    return 0;
}
```

### Règle d'or

> **En cas de doute, utilisez des parenthèses !**
> 
> Le code explicite est toujours préférable au code "malin".

---

## Comparaison avec Python/JS/C#/Go

### Opérateurs présents dans tous les langages

| Opérateur         | C  | Python           | JavaScript  | C# | Go               |
|-------------------|----|------------------|-------------|----|------------------|
| `+` `-` `*` `/`   | ✅ | ✅              | ✅         | ✅ | ✅               |
| `%` (modulo)      | ✅ | ✅              | ✅         | ✅ | ✅               |
| `==` `!=`         | ✅ | ✅              | ✅ (`===`) | ✅ | ✅               | 
| `<` `>` `<=` `>=` | ✅ | ✅              | ✅         | ✅ | ✅               |
| `&&` `\|\|` `!`   | ✅ | `and` `or` `not` | ✅        | ✅ | ✅               |
| `&` `\|` `^` `~`  | ✅ | ✅              | ✅         | ✅ | ✅               |
| `<<` `>>`         | ✅ | ✅              | ✅         | ✅ | ✅               |
| `++` `--`         | ✅ | ❌              | ✅         | ✅ | ✅ (instruction) |
| `? :`             | ✅ | `x if c else y` | ✅         | ✅ | ❌               |

### Opérateurs spécifiques

```c
// C : sizeof (opérateur)
size_t s = sizeof(int);
```

```python
# Python : puissance **, division entière //, walrus :=
x = 2 ** 10  # 1024
y = 7 // 3   # 2
if (n := len(data)) > 10:
    print(n)
```

```javascript
// JavaScript : égalité stricte ===, nullish coalescing ??
if (x === 5) { }
let val = x ?? defaultValue;
let y = obj?.property;  // Optional chaining
```

```go
// Go : pas de ternaire, := pour déclaration courte
x := 42  // Déclaration + initialisation
// max := (a > b) ? a : b  // ERREUR: pas de ternaire en Go
```

### Logique booléenne : Python vs C

```python
# Python : mots-clés, valeurs truthy/falsy
if x and y or not z:
    pass

# Valeurs falsy : None, False, 0, 0.0, "", [], {}, set()
```

```c
// C : symboles, toute valeur non-nulle est vraie
if (x && y || !z) {
    // ...
}

// Valeurs fausses : 0, 0.0, NULL
// Tout le reste est vrai (y compris "string" car c'est un pointeur non nul)
```

---

## Exercices du chapitre

### Exercice 3.1 : Division et modulo (Débutant)

Écrivez un programme qui :
1. Demande deux entiers à l'utilisateur
2. Affiche le quotient et le reste de la division
3. Affiche le résultat de la division flottante

```
Entrez a : 17
Entrez b : 5
Division entière : 17 / 5 = 3 reste 2
Division flottante : 17 / 5 = 3.400000
```

---

### Exercice 3.2 : Parité et divisibilité (Débutant)

Créez un programme qui demande un entier et affiche :
- S'il est pair ou impair (utilisez `%`)
- S'il est divisible par 3
- S'il est divisible par 5
- S'il est divisible par 3 ET par 5 (FizzBuzz!)

---

### Exercice 3.3 : Calculatrice avec opérateurs (Intermédiaire)

Créez une calculatrice qui :
1. Demande deux nombres et un opérateur (+, -, *, /, %)
2. Effectue l'opération et affiche le résultat
3. Gère la division par zéro
4. Gère les opérateurs invalides

---

### Exercice 3.4 : Manipulation de bits - Extraire des octets (Intermédiaire)

Un entier 32 bits contient 4 octets. Écrivez un programme qui :
1. Prend un entier en entrée
2. Affiche chaque octet séparément (en décimal et hexadécimal)

```
Entrez un nombre : 305419896
Octet 0 (LSB) : 120 (0x78)
Octet 1       : 86  (0x56)
Octet 2       : 52  (0x34)
Octet 3 (MSB) : 18  (0x12)
```

**Indice** : Utilisez des masques et des décalages.

---

### Exercice 3.5 : Permutation circulaire de bits (Intermédiaire)

Implémentez deux fonctions :
- `rotate_left(unsigned int x, int n)` : rotation gauche de n bits
- `rotate_right(unsigned int x, int n)` : rotation droite de n bits

Une rotation est comme un décalage, mais les bits qui "sortent" réapparaissent de l'autre côté.

```c
// Exemple : 0b10110001 rotation gauche de 3
// Résultat : 0b10001101
```

---

### Exercice 3.6 : Encodeur/Décodeur XOR (Intermédiaire)

Créez un programme qui :
1. Demande une chaîne de caractères
2. Demande une clé (entier)
3. Chiffre la chaîne avec XOR
4. Déchiffre pour vérifier

---

### Exercice 3.7 : Compter les bits (Intermédiaire)

Implémentez plusieurs méthodes pour compter le nombre de bits à 1 dans un entier :

1. Méthode naïve (parcourir tous les bits)
2. Méthode de Brian Kernighan (`n & (n-1)` élimine le bit 1 le plus à droite)
3. Comparer les performances

---

### Exercice 3.8 : Drapeaux de permissions (Intermédiaire-Avancé)

Créez un système de permissions façon UNIX avec les flags :

```c
#define PERM_READ    0x04  // r--
#define PERM_WRITE   0x02  // -w-
#define PERM_EXECUTE 0x01  // --x
```

Implémentez :
- `set_permission(flags, perm)` : ajoute une permission
- `clear_permission(flags, perm)` : retire une permission
- `has_permission(flags, perm)` : vérifie une permission
- `display_permissions(flags)` : affiche comme "rwx" ou "r-x" etc.

---

### Exercice 3.9 : Évaluation d'expressions (Avancé)

Sans exécuter le code, déterminez la valeur de chaque expression, puis vérifiez :

```c
int a = 5, b = 3, c = 2;

int r1 = a + b * c;
int r2 = (a + b) * c;
int r3 = a++ + ++b;
int r4 = a > b ? a : b;
int r5 = a & b | c;
int r6 = a && b || c;
int r7 = !a + !b;
int r8 = a << 2 + b;  // Piège!
```

---

### Exercice 3.10 : Mini-projet : Convertisseur de couleurs (Avancé)

Une couleur RGB est souvent stockée dans un seul entier 32 bits : `0x00RRGGBB`

Implémentez :
1. `int rgb_to_int(int r, int g, int b)` : combine 3 valeurs 0-255
2. `void int_to_rgb(int color, int *r, int *g, int *b)` : extrait les composantes
3. `int blend(int color1, int color2, float ratio)` : mélange deux couleurs
4. `int invert(int color)` : inverse une couleur
5. `int grayscale(int color)` : convertit en niveaux de gris

---

## Résumé du chapitre

✅ **Opérateurs arithmétiques** : `+`, `-`, `*`, `/`, `%` - Attention à la division entière !

✅ **Opérateurs de comparaison** : `==`, `!=`, `<`, `>`, `<=`, `>=` - Retournent 0 ou 1

✅ **Opérateurs logiques** : `&&`, `||`, `!` - Évaluation en court-circuit

✅ **Opérateurs bit-à-bit** : `&`, `|`, `^`, `~`, `<<`, `>>` - Travaillent sur les bits individuels

✅ **Affectations composées** : `+=`, `-=`, `&=`, `|=`, etc.

✅ **Incrémentation** : `++x` (pré) vs `x++` (post) - Une seule modification par expression !

✅ **Ternaire** : `condition ? vrai : faux` - Alternative compacte à if-else

✅ **Précédence** : Multiplication avant addition, AND avant OR - En cas de doute, parenthèses !

✅ **Pièges courants** : `=` vs `==`, comparaison signed/unsigned, débordements

---

**Prochain chapitre** : Structures de contrôle (conditions et boucles)
