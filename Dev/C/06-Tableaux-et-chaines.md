# Chapitre 6 : Tableaux et chaînes de caractères

## Table des matières

1. [Tableaux unidimensionnels](#tableaux-unidimensionnels)
2. [Tableaux multidimensionnels](#tableaux-multidimensionnels)
3. [Tableaux et pointeurs](#tableaux-et-pointeurs)
4. [Chaînes de caractères](#chaînes-de-caractères)
5. [Fonctions de manipulation de chaînes](#fonctions-de-manipulation-de-chaînes)
6. [Tableaux de chaînes](#tableaux-de-chaînes)
7. [Sécurité et bonnes pratiques](#sécurité-et-bonnes-pratiques)
8. [Comparaison avec Python/JS/C#/Go](#comparaison-avec-pythonjscgo)
9. [Exercices du chapitre](#exercices-du-chapitre)

---

## Tableaux unidimensionnels

### Déclaration et initialisation

```c
#include <stdio.h>

int main(void) {
    // Déclaration avec taille explicite
    int nombres[5];  // 5 entiers, valeurs indéterminées!
    
    // Déclaration avec initialisation
    int notes[5] = {15, 12, 18, 14, 16};
    
    // Initialisation partielle (reste = 0)
    int partiels[5] = {1, 2};  // {1, 2, 0, 0, 0}
    
    // Taille implicite
    int auto_taille[] = {10, 20, 30};  // Taille = 3
    
    // Tout à zéro
    int zeros[100] = {0};  // Tous les éléments à 0
    
    // Taille du tableau
    int taille = sizeof(notes) / sizeof(notes[0]);
    printf("Taille : %d\n", taille);  // 5
    
    return 0;
}
```

### Accès aux éléments

```c
#include <stdio.h>

int main(void) {
    int arr[5] = {10, 20, 30, 40, 50};
    
    // Lecture
    printf("Premier : %d\n", arr[0]);   // 10
    printf("Dernier : %d\n", arr[4]);   // 50
    
    // Modification
    arr[2] = 35;
    
    // Parcours avec for
    for (int i = 0; i < 5; i++) {
        printf("arr[%d] = %d\n", i, arr[i]);
    }
    
    // DANGER : pas de vérification des bornes!
    // arr[10] = 100;  // Buffer overflow! Comportement indéfini
    // arr[-1] = 0;    // Également indéfini
    
    return 0;
}
```

### Opérations courantes

```c
#include <stdio.h>

int main(void) {
    int arr[] = {5, 2, 8, 1, 9, 3, 7, 4, 6};
    int n = sizeof(arr) / sizeof(arr[0]);
    
    // Somme
    int somme = 0;
    for (int i = 0; i < n; i++) {
        somme += arr[i];
    }
    printf("Somme : %d\n", somme);
    
    // Moyenne
    double moyenne = (double)somme / n;
    printf("Moyenne : %.2f\n", moyenne);
    
    // Min et Max
    int min = arr[0], max = arr[0];
    for (int i = 1; i < n; i++) {
        if (arr[i] < min) min = arr[i];
        if (arr[i] > max) max = arr[i];
    }
    printf("Min : %d, Max : %d\n", min, max);
    
    // Recherche
    int cible = 8;
    int index = -1;
    for (int i = 0; i < n; i++) {
        if (arr[i] == cible) {
            index = i;
            break;
        }
    }
    if (index >= 0) {
        printf("%d trouvé à l'index %d\n", cible, index);
    }
    
    // Inversion
    for (int i = 0; i < n / 2; i++) {
        int temp = arr[i];
        arr[i] = arr[n - 1 - i];
        arr[n - 1 - i] = temp;
    }
    
    return 0;
}
```

---

## Tableaux multidimensionnels

### Tableaux 2D (matrices)

```c
#include <stdio.h>

int main(void) {
    // Déclaration : [lignes][colonnes]
    int matrice[3][4] = {
        {1, 2, 3, 4},      // Ligne 0
        {5, 6, 7, 8},      // Ligne 1
        {9, 10, 11, 12}    // Ligne 2
    };
    
    // Accès
    printf("Element [1][2] : %d\n", matrice[1][2]);  // 7
    
    // Parcours
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 4; j++) {
            printf("%3d ", matrice[i][j]);
        }
        printf("\n");
    }
    
    // Initialisation à zéro
    int zeros[10][10] = {{0}};
    
    // Matrice identité
    int identite[3][3] = {
        {1, 0, 0},
        {0, 1, 0},
        {0, 0, 1}
    };
    
    return 0;
}
```

### Opérations sur matrices

```c
#include <stdio.h>

#define LIGNES 3
#define COLS 3

void afficher_matrice(int m[LIGNES][COLS]) {
    for (int i = 0; i < LIGNES; i++) {
        for (int j = 0; j < COLS; j++) {
            printf("%4d ", m[i][j]);
        }
        printf("\n");
    }
}

void addition_matrices(int a[LIGNES][COLS], int b[LIGNES][COLS], int resultat[LIGNES][COLS]) {
    for (int i = 0; i < LIGNES; i++) {
        for (int j = 0; j < COLS; j++) {
            resultat[i][j] = a[i][j] + b[i][j];
        }
    }
}

void transposer(int m[LIGNES][COLS], int transposee[COLS][LIGNES]) {
    for (int i = 0; i < LIGNES; i++) {
        for (int j = 0; j < COLS; j++) {
            transposee[j][i] = m[i][j];
        }
    }
}

int main(void) {
    int a[3][3] = {{1,2,3}, {4,5,6}, {7,8,9}};
    int b[3][3] = {{9,8,7}, {6,5,4}, {3,2,1}};
    int somme[3][3];
    
    addition_matrices(a, b, somme);
    printf("Somme :\n");
    afficher_matrice(somme);
    
    return 0;
}
```

---

## Tableaux et pointeurs

### Relation fondamentale

En C, le nom d'un tableau est un **pointeur constant** vers son premier élément :

```c
#include <stdio.h>

int main(void) {
    int arr[] = {10, 20, 30, 40, 50};
    
    // arr est équivalent à &arr[0]
    printf("arr       = %p\n", (void*)arr);
    printf("&arr[0]   = %p\n", (void*)&arr[0]);
    
    // Arithmétique des pointeurs
    int *p = arr;
    printf("*p       = %d\n", *p);        // 10
    printf("*(p+1)   = %d\n", *(p + 1));  // 20
    printf("*(p+2)   = %d\n", *(p + 2));  // 30
    
    // Équivalences
    printf("arr[2]   = %d\n", arr[2]);    // 30
    printf("*(arr+2) = %d\n", *(arr + 2)); // 30
    printf("2[arr]   = %d\n", 2[arr]);    // 30 (!bizarre mais valide)
    
    // Parcours avec pointeur
    for (int *ptr = arr; ptr < arr + 5; ptr++) {
        printf("%d ", *ptr);
    }
    printf("\n");
    
    return 0;
}
```

### Passage de tableau à une fonction

```c
#include <stdio.h>

// Ces trois déclarations sont équivalentes pour les paramètres :
void fonction1(int arr[]);      // Syntaxe tableau
void fonction2(int arr[10]);    // La taille est ignorée!
void fonction3(int *arr);       // Syntaxe pointeur (explicite)

void afficher(int *arr, int taille) {
    for (int i = 0; i < taille; i++) {
        printf("%d ", arr[i]);  // ou *(arr + i)
    }
    printf("\n");
}

void modifier(int *arr, int taille) {
    for (int i = 0; i < taille; i++) {
        arr[i] *= 2;  // Modifie le tableau original!
    }
}

int main(void) {
    int tab[] = {1, 2, 3, 4, 5};
    int n = sizeof(tab) / sizeof(tab[0]);
    
    afficher(tab, n);  // 1 2 3 4 5
    modifier(tab, n);
    afficher(tab, n);  // 2 4 6 8 10
    
    return 0;
}
```

---

## Chaînes de caractères

### Définition

En C, une chaîne est un **tableau de caractères terminé par `'\0'`** (caractère nul) :

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    // Différentes façons de déclarer
    char str1[] = "Hello";           // Tableau modifiable, taille 6
    char str2[10] = "Hello";         // Tableau avec espace supplémentaire
    char str3[] = {'H','e','l','l','o','\0'};  // Équivalent explicite
    const char *str4 = "Hello";      // Pointeur vers littéral (NON modifiable)
    
    // Attention à la taille!
    printf("sizeof(str1) = %zu\n", sizeof(str1));  // 6 (5 + '\0')
    printf("strlen(str1) = %zu\n", strlen(str1));  // 5 (sans '\0')
    
    // Représentation en mémoire de "Hi" :
    // +---+---+---+
    // | H | i |\0 |
    // +---+---+---+
    // [0] [1] [2]
    
    // Affichage
    printf("%s\n", str1);  // printf s'arrête au '\0'
    
    // Caractère par caractère
    for (int i = 0; str1[i] != '\0'; i++) {
        printf("[%d] = '%c' (ASCII %d)\n", i, str1[i], str1[i]);
    }
    
    return 0;
}
```

### Lecture de chaînes

```c
#include <stdio.h>

int main(void) {
    char nom[50];
    char ligne[100];
    
    // scanf avec %s : lit jusqu'au whitespace (DANGEREUX!)
    printf("Votre prénom : ");
    scanf("%49s", nom);  // Limiter à 49 chars + '\0'
    
    // fgets : lit une ligne entière (RECOMMANDÉ)
    printf("Une phrase : ");
    getchar();  // Consomme le '\n' restant de scanf
    fgets(ligne, sizeof(ligne), stdin);
    
    // Supprimer le '\n' de fgets
    ligne[strcspn(ligne, "\n")] = '\0';
    
    printf("Prénom : %s\n", nom);
    printf("Phrase : %s\n", ligne);
    
    // JAMAIS utiliser gets() - retiré depuis C11
    // gets(ligne);  // EXTRÊMEMENT DANGEREUX!
    
    return 0;
}
```

---

## Fonctions de manipulation de chaînes

### Bibliothèque `<string.h>`

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char src[] = "Hello";
    char dest[20];
    char str1[50] = "Bon";
    char str2[] = "jour";
    
    // strlen : longueur (sans '\0')
    printf("strlen(src) = %zu\n", strlen(src));  // 5
    
    // strcpy : copie (DANGEREUX - pas de vérification de taille)
    strcpy(dest, src);  // dest = "Hello"
    
    // strncpy : copie sécurisée avec limite
    strncpy(dest, src, sizeof(dest) - 1);
    dest[sizeof(dest) - 1] = '\0';  // Garantir terminaison
    
    // strcat : concaténation (DANGEREUX)
    strcat(str1, str2);  // str1 = "Bonjour"
    
    // strncat : concaténation sécurisée
    strncat(str1, "!", sizeof(str1) - strlen(str1) - 1);
    
    // strcmp : comparaison
    int cmp = strcmp("abc", "abd");
    // < 0 si s1 < s2
    // = 0 si s1 == s2
    // > 0 si s1 > s2
    printf("strcmp : %d\n", cmp);  // < 0
    
    // strncmp : comparaison des n premiers caractères
    printf("strncmp(\"Hello\", \"Help\", 3) = %d\n", 
           strncmp("Hello", "Help", 3));  // 0
    
    // strchr : cherche un caractère
    char *pos = strchr("Hello", 'l');
    if (pos) printf("'l' trouvé à position %ld\n", pos - "Hello");
    
    // strstr : cherche une sous-chaîne
    char *sub = strstr("Hello World", "Wor");
    if (sub) printf("Sous-chaîne trouvée : %s\n", sub);
    
    return 0;
}
```

### Implémentation manuelle des fonctions

```c
#include <stdio.h>

// strlen
size_t ma_strlen(const char *str) {
    size_t len = 0;
    while (str[len] != '\0') {
        len++;
    }
    return len;
}

// strcpy
char *ma_strcpy(char *dest, const char *src) {
    char *original = dest;
    while ((*dest++ = *src++) != '\0');
    return original;
}

// strcmp
int ma_strcmp(const char *s1, const char *s2) {
    while (*s1 && (*s1 == *s2)) {
        s1++;
        s2++;
    }
    return *(unsigned char*)s1 - *(unsigned char*)s2;
}

// strcat
char *ma_strcat(char *dest, const char *src) {
    char *original = dest;
    while (*dest) dest++;  // Aller à la fin
    while ((*dest++ = *src++) != '\0');
    return original;
}

// strchr
char *ma_strchr(const char *str, int c) {
    while (*str) {
        if (*str == (char)c) {
            return (char*)str;
        }
        str++;
    }
    return (c == '\0') ? (char*)str : NULL;
}
```

### Autres fonctions utiles

```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>
#include <stdlib.h>

int main(void) {
    // Conversion casse
    char str[] = "Hello World";
    for (int i = 0; str[i]; i++) {
        str[i] = toupper(str[i]);  // ou tolower()
    }
    printf("Majuscules : %s\n", str);
    
    // Conversion nombre ↔ chaîne
    int n = atoi("42");           // Chaîne → int
    double d = atof("3.14159");   // Chaîne → double
    long l = strtol("1000", NULL, 10);  // Plus sûr
    
    char buffer[50];
    sprintf(buffer, "%d", n);     // int → chaîne
    printf("Buffer : %s\n", buffer);
    
    // snprintf (sécurisé)
    snprintf(buffer, sizeof(buffer), "Valeur : %d", 42);
    
    // Tokenisation
    char phrase[] = "un,deux,trois,quatre";
    char *token = strtok(phrase, ",");
    while (token != NULL) {
        printf("Token : %s\n", token);
        token = strtok(NULL, ",");
    }
    
    return 0;
}
```

---

## Tableaux de chaînes

```c
#include <stdio.h>

int main(void) {
    // Tableau de pointeurs vers chaînes littérales
    const char *jours[] = {
        "Lundi", "Mardi", "Mercredi", "Jeudi",
        "Vendredi", "Samedi", "Dimanche"
    };
    
    // Tableau 2D de caractères
    char noms[3][20] = {
        "Alice",
        "Bob",
        "Charlie"
    };
    
    // Affichage
    for (int i = 0; i < 7; i++) {
        printf("%s\n", jours[i]);
    }
    
    // Modification possible avec tableau 2D
    noms[0][0] = 'E';  // "Alice" → "Elice"
    
    // Mais pas avec pointeurs vers littéraux!
    // jours[0][0] = 'X';  // CRASH! Littéral en mémoire read-only
    
    // Arguments de main
    // int main(int argc, char *argv[])
    // argv est un tableau de chaînes
    
    return 0;
}
```

---

## Sécurité et bonnes pratiques

### Vulnérabilités courantes

```c
#include <stdio.h>
#include <string.h>

// MAUVAIS : Buffer overflow avec strcpy
void dangereux(void) {
    char buffer[10];
    strcpy(buffer, "Cette chaîne est beaucoup trop longue!");
    // Débordement! Écrase la mémoire adjacente
}

// BON : Utiliser strncpy
void securise(const char *input) {
    char buffer[10];
    strncpy(buffer, input, sizeof(buffer) - 1);
    buffer[sizeof(buffer) - 1] = '\0';
}

// MAUVAIS : gets (retiré du standard)
void tres_dangereux(void) {
    char buffer[100];
    // gets(buffer);  // JAMAIS! Pas de limite de taille
}

// BON : fgets
void lecture_securisee(void) {
    char buffer[100];
    if (fgets(buffer, sizeof(buffer), stdin) != NULL) {
        buffer[strcspn(buffer, "\n")] = '\0';
    }
}

// MAUVAIS : sprintf sans limite
void format_dangereux(int n) {
    char buffer[10];
    sprintf(buffer, "Nombre: %d", n);  // Peut déborder
}

// BON : snprintf
void format_securise(int n) {
    char buffer[10];
    snprintf(buffer, sizeof(buffer), "N: %d", n);
}
```

### Règles d'or

1. **Toujours spécifier les tailles** : `strncpy`, `strncat`, `snprintf`, `fgets`
2. **Vérifier les retours** : `fgets`, `malloc` peuvent échouer
3. **Initialiser les buffers** : `char buffer[100] = {0};`
4. **Garantir la terminaison** : Après `strncpy`, ajouter `'\0'`
5. **Valider les entrées** : Vérifier la longueur avant copie

---

## Comparaison avec Python/JS/C#/Go

| Aspect        | C          | Python      | JavaScript  | Go          |
|---------------|------------|-------------|-------------|-------------|
| Type chaîne   | `char[]`   | `str`       | `String`    | `string`    |
| Mutabilité    | Mutable    | Immuable    | Immuable    | Immuable    |
| Terminaison   | `\0`       | Implicite   | Implicite   | Implicite   |
| Longueur      | `strlen()` | `len()`     | `.length`   | `len()`     |
| Concaténation | `strcat()` | `+`         | `+`         | `+`         |
| Sécurité      | Manuelle   | Automatique | Automatique | Automatique |

```python
# Python : simple et sûr
s = "Hello"
s += " World"  # Crée une nouvelle chaîne
print(len(s))
```

```c
// C : contrôle total mais dangereux
char s[50] = "Hello";
strcat(s, " World");  // Modifie en place, risque de débordement
printf("%zu\n", strlen(s));
```

---

## Exercices du chapitre

### Exercice 6.1 : Statistiques tableau (Débutant)
Créez des fonctions pour calculer somme, moyenne, min, max, écart-type.

### Exercice 6.2 : Rotation de tableau (Intermédiaire)
Implémentez la rotation à gauche et à droite de n positions.

### Exercice 6.3 : Tri (Intermédiaire)
Implémentez tri à bulles, tri par sélection, tri par insertion.

### Exercice 6.4 : Manipulation de chaînes (Intermédiaire)
Réimplémentez `strlen`, `strcpy`, `strcmp`, `strcat`, `strchr`, `strstr`.

### Exercice 6.5 : Palindrome (Intermédiaire)
Vérifiez si une chaîne est un palindrome (ignorez casse et espaces).

### Exercice 6.6 : Matrices (Intermédiaire-Avancé)
Implémentez addition, multiplication, transposition de matrices.

### Exercice 6.7 : Anagramme (Avancé)
Vérifiez si deux chaînes sont des anagrammes.

### Exercice 6.8 : Parser CSV (Avancé)
Parsez une ligne CSV et extrayez les champs dans un tableau de chaînes.

---

## Résumé du chapitre

**Tableaux** : Collections de taille fixe, indexées à partir de 0

**Pas de bounds checking** : C ne vérifie pas les accès hors limites

**Tableaux = pointeurs** : Le nom d'un tableau est un pointeur vers son premier élément

**Chaînes** : Tableaux de `char` terminés par `'\0'`

**Fonctions sécurisées** : Préférer `strncpy`, `strncat`, `snprintf`, `fgets`

**Attention aux débordements** : Source majeure de vulnérabilités

---

**Prochain chapitre** : Pointeurs
