# Chapitre 9 : Gestion de la mémoire et fichiers

## Table des matières

1.  [Modèle mémoire en C](#modèle-mémoire-en-c)
2.  [Allocation dynamique](#allocation-dynamique)
3.  [malloc, calloc, realloc, free](#malloc-calloc-realloc-free)
4.  [Erreurs mémoire courantes](#erreurs-mémoire-courantes)
5.  [Outils de débogage mémoire](#outils-de-débogage-mémoire)
6.  [Manipulation de fichiers](#manipulation-de-fichiers)
7.  [Fichiers texte](#fichiers-texte)
8.  [Fichiers binaires](#fichiers-binaires)
9.  [Gestion des erreurs](#gestion-des-erreurs)
10. [Bonnes pratiques](#bonnes-pratiques)
11. [Comparaison avec Python/JS/C#/Go](#comparaison-avec-pythonjscgo)
12. [Exercices du chapitre](#exercices-du-chapitre)

---

## Modèle mémoire en C

### Les quatre segments de mémoire

```
┌─────────────────────────────────┐  Adresses hautes
│           STACK                 │  ← Variables locales, paramètres
│         (croît vers le bas)     │    Taille limitée (~1-8 MB)
│              ↓                  │
├─────────────────────────────────┤
│                                 │
│         (espace libre)          │
│                                 │
├─────────────────────────────────┤
│              ↑                  │
│         (croît vers le haut)    │
│           HEAP                  │  ← Allocation dynamique (malloc)
├─────────────────────────────────┤
│           BSS                   │  ← Variables globales non initialisées
├─────────────────────────────────┤
│           DATA                  │  ← Variables globales initialisées
├─────────────────────────────────┤
│           TEXT                  │  ← Code du programme (lecture seule)
└─────────────────────────────────┘  Adresses basses
```

### Exemple pratique

```c
#include <stdio.h>
#include <stdlib.h>

int global_init = 42;        // Segment DATA
int global_non_init;         // Segment BSS (initialisé à 0)

void fonction(int param) {   // param sur la STACK
    int local = 10;          // local sur la STACK
    static int statique = 5; // Segment DATA (persiste)
    
    int *heap = malloc(sizeof(int));  // Pointeur sur STACK, données sur HEAP
    *heap = 100;
    
    printf("param: %p (stack)\n", (void*)&param);
    printf("local: %p (stack)\n", (void*)&local);
    printf("heap:  %p (heap)\n", (void*)heap);
    
    free(heap);
}

int main(void) {
    printf("global_init: %p (data)\n", (void*)&global_init);
    printf("global_non_init: %p (bss)\n", (void*)&global_non_init);
    printf("main: %p (text)\n", (void*)main);
    
    fonction(5);
    return 0;
}
```

### Durée de vie des variables

| Type          | Allocation              | Libération            | Durée de vie           |
|---------------|-------------------------|-----------------------|------------------------|
| Locale (auto) | Entrée dans la fonction | Sortie de la fonction | Courte                 |
| Static        | Démarrage programme     | Fin programme         | Programme entier       |
| Globale       | Démarrage programme     | Fin programme         | Programme entier       |
| Dynamique     | `malloc()`              | `free()`              | Contrôlée manuellement |

---

## Allocation dynamique

### Pourquoi l'allocation dynamique ?

1. **Taille inconnue à la compilation** : L'utilisateur entre le nombre d'éléments
2. **Structures de données flexibles** : Listes chaînées, arbres
3. **Données volumineuses** : La stack est limitée (~1-8 MB)
4. **Durée de vie contrôlée** : Allocation et libération à la demande

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int n;
    printf("Combien d'éléments ? ");
    scanf("%d", &n);
    
    // Impossible avec un tableau statique :
    // int tableau[n];  // VLA, non supporté partout, taille limitée
    
    // Solution : allocation dynamique
    int *tableau = malloc(n * sizeof(int));
    if (tableau == NULL) {
        fprintf(stderr, "Erreur d'allocation\n");
        return 1;
    }
    
    // Utilisation comme un tableau normal
    for (int i = 0; i < n; i++) {
        tableau[i] = i * 10;
    }
    
    // Libération obligatoire
    free(tableau);
    
    return 0;
}
```

---

## malloc, calloc, realloc, free

### malloc : allocation simple

```c
#include <stdlib.h>

// void *malloc(size_t size)
// Alloue 'size' octets, contenu NON initialisé (garbage)

int *entier = malloc(sizeof(int));
int *tableau = malloc(100 * sizeof(int));
char *chaine = malloc(256);  // sizeof(char) = 1

// TOUJOURS vérifier le retour
if (tableau == NULL) {
    // Gestion d'erreur
}
```

### calloc : allocation initialisée à zéro

```c
// void *calloc(size_t nmemb, size_t size)
// Alloue nmemb éléments de taille size, INITIALISÉS À ZÉRO

int *tableau = calloc(100, sizeof(int));  // 100 int, tous à 0

// Équivalent à :
int *tableau2 = malloc(100 * sizeof(int));
memset(tableau2, 0, 100 * sizeof(int));
```

### realloc : redimensionnement

```c
// void *realloc(void *ptr, size_t size)
// Redimensionne la zone mémoire, peut déplacer les données

int *arr = malloc(10 * sizeof(int));
// ... remplir arr avec des données ...

// Agrandir à 20 éléments
int *new_arr = realloc(arr, 20 * sizeof(int));
if (new_arr == NULL) {
    // Erreur : arr est TOUJOURS valide!
    free(arr);
    return;
}
arr = new_arr;  // Utiliser le nouveau pointeur

// Cas spéciaux :
realloc(NULL, size);  // Équivalent à malloc(size)
realloc(ptr, 0);      // Équivalent à free(ptr), retourne NULL
```

### free : libération

```c
// void free(void *ptr)
// Libère la mémoire allouée par malloc/calloc/realloc

int *p = malloc(sizeof(int));
*p = 42;
free(p);      // Libère la mémoire

// Bonne pratique : mettre à NULL après free
p = NULL;     // Évite use-after-free et double free
free(NULL);   // Sûr, ne fait rien
```

### Pattern d'allocation sécurisée

```c
#include <stdio.h>
#include <stdlib.h>

// Wrapper qui gère l'erreur
void *safe_malloc(size_t size) {
    if (size == 0) return NULL;
    
    void *ptr = malloc(size);
    if (ptr == NULL) {
        fprintf(stderr, "Erreur: allocation de %zu octets échouée\n", size);
        exit(EXIT_FAILURE);
    }
    return ptr;
}

// Wrapper pour free qui met à NULL
void safe_free(void **ptr) {
    if (ptr != NULL && *ptr != NULL) {
        free(*ptr);
        *ptr = NULL;
    }
}

// Utilisation
int main(void) {
    int *arr = safe_malloc(100 * sizeof(int));
    // ... utilisation ...
    safe_free((void**)&arr);  // arr est maintenant NULL
    return 0;
}
```

---

## Erreurs mémoire courantes

### 1. Fuite mémoire (Memory Leak)

```c
void fuite(void) {
    int *p = malloc(1000);
    // Oubli de free(p)!
}  // p perdu, mémoire jamais libérée

// Dans une boucle, c'est catastrophique :
for (int i = 0; i < 1000000; i++) {
    int *p = malloc(1000);
    // Fuite : 1 Go de mémoire perdue!
}
```

### 2. Use-After-Free

```c
int *p = malloc(sizeof(int));
*p = 42;
free(p);
*p = 100;  // DANGER! Mémoire libérée

// Solution : p = NULL après free
```

### 3. Double Free

```c
int *p = malloc(sizeof(int));
free(p);
free(p);  // CRASH! Double libération

// Solution : p = NULL après free
```

### 4. Buffer Overflow

```c
int *arr = malloc(10 * sizeof(int));
arr[10] = 42;  // DANGER! Écriture hors limites

// Solution : vérifier les indices
```

### 5. Lecture non initialisée

```c
int *p = malloc(sizeof(int));
printf("%d\n", *p);  // DANGER! Valeur indéterminée

// Solution : calloc ou initialisation explicite
```

### 6. Pointeur invalide

```c
int *p;  // Non initialisé
*p = 42; // CRASH!

// Solution : initialiser à NULL ou à une adresse valide
```

---

## Outils de débogage mémoire

### Valgrind

```bash
# Compilation avec symboles de débogage
gcc -g -O0 programme.c -o programme

# Exécution avec Valgrind
valgrind --leak-check=full --show-leak-kinds=all ./programme
```

Sortie exemple :
```
==12345== HEAP SUMMARY:
==12345==     in use at exit: 1,000 bytes in 1 blocks
==12345==   total heap usage: 10 allocs, 9 frees, 10,000 bytes allocated
==12345== 
==12345== 1,000 bytes in 1 blocks are definitely lost in loss record 1 of 1
==12345==    at 0x4C2BBAF: malloc (vg_replace_malloc.c:299)
==12345==    by 0x4005E7: fonction_fuite (programme.c:15)
==12345==    by 0x400627: main (programme.c:25)
```

### AddressSanitizer (ASan)

```bash
# Compilation avec ASan
gcc -fsanitize=address -g programme.c -o programme

# Exécution normale
./programme
```

Détecte automatiquement :
- Buffer overflow (heap et stack)
- Use-after-free
- Double free
- Memory leaks

### MemorySanitizer (MSan)

```bash
# Avec Clang
clang -fsanitize=memory -g programme.c -o programme
```

Détecte les lectures de mémoire non initialisée.

---

## Manipulation de fichiers

### Ouverture et fermeture

```c
#include <stdio.h>

int main(void) {
    // Ouverture
    FILE *fichier = fopen("data.txt", "r");
    
    // TOUJOURS vérifier
    if (fichier == NULL) {
        perror("Erreur ouverture");
        return 1;
    }
    
    // ... opérations sur le fichier ...
    
    // Fermeture obligatoire
    fclose(fichier);
    
    return 0;
}
```

### Modes d'ouverture

| Mode   | Description      | Fichier existe     | Fichier n'existe pas |
|--------|------------------|--------------------|----------------------|
| `"r"`  | Lecture          | OK                 | Erreur               |
| `"w"`  | Écriture         | Écrasé             | Créé                 |
| `"a"`  | Ajout            | Ajout à la fin     | Créé                 |
| `"r+"` | Lecture/Écriture | OK                 | Erreur               |
| `"w+"` | Lecture/Écriture | Écrasé             | Créé                 |
| `"a+"` | Lecture/Ajout    | OK, ajout à la fin | Créé                 |
| `"rb"` | Lecture binaire  |          -         |           -          |
| `"wb"` | Écriture binaire |          -         |           -          |

---

## Fichiers texte

### Lecture

```c
#include <stdio.h>

int main(void) {
    FILE *f = fopen("data.txt", "r");
    if (f == NULL) {
        perror("Erreur");
        return 1;
    }
    
    // Méthode 1 : fgetc (caractère par caractère)
    int c;
    while ((c = fgetc(f)) != EOF) {
        putchar(c);
    }
    rewind(f);  // Retour au début
    
    // Méthode 2 : fgets (ligne par ligne) - RECOMMANDÉ
    char ligne[256];
    while (fgets(ligne, sizeof(ligne), f) != NULL) {
        printf("%s", ligne);  // ligne contient le \n
    }
    rewind(f);
    
    // Méthode 3 : fscanf (formaté)
    int n;
    float x;
    while (fscanf(f, "%d %f", &n, &x) == 2) {
        printf("Lu : %d, %.2f\n", n, x);
    }
    
    fclose(f);
    return 0;
}
```

### Écriture

```c
#include <stdio.h>

int main(void) {
    FILE *f = fopen("sortie.txt", "w");
    if (f == NULL) {
        perror("Erreur");
        return 1;
    }
    
    // fputc : un caractère
    fputc('A', f);
    fputc('\n', f);
    
    // fputs : une chaîne (sans \n automatique)
    fputs("Bonjour le monde\n", f);
    
    // fprintf : formaté (comme printf)
    for (int i = 0; i < 5; i++) {
        fprintf(f, "Ligne %d : valeur = %d\n", i, i * 10);
    }
    
    fclose(f);
    return 0;
}
```

### Exemple : copie de fichier

```c
#include <stdio.h>

int copier_fichier(const char *source, const char *destination) {
    FILE *src = fopen(source, "r");
    if (src == NULL) {
        perror("Erreur source");
        return -1;
    }
    
    FILE *dst = fopen(destination, "w");
    if (dst == NULL) {
        perror("Erreur destination");
        fclose(src);
        return -1;
    }
    
    char buffer[4096];
    size_t bytes_lus;
    
    while ((bytes_lus = fread(buffer, 1, sizeof(buffer), src)) > 0) {
        fwrite(buffer, 1, bytes_lus, dst);
    }
    
    fclose(src);
    fclose(dst);
    return 0;
}
```

---

## Fichiers binaires

### Écriture binaire

```c
#include <stdio.h>

typedef struct {
    char nom[50];
    int age;
    float salaire;
} Employe;

int main(void) {
    Employe emp = {"Alice", 30, 45000.0};
    
    FILE *f = fopen("employes.bin", "wb");
    if (f == NULL) {
        perror("Erreur");
        return 1;
    }
    
    // Écrire une structure
    fwrite(&emp, sizeof(Employe), 1, f);
    
    // Écrire un tableau
    int tableau[] = {1, 2, 3, 4, 5};
    fwrite(tableau, sizeof(int), 5, f);
    
    fclose(f);
    return 0;
}
```

### Lecture binaire

```c
#include <stdio.h>

typedef struct {
    char nom[50];
    int age;
    float salaire;
} Employe;

int main(void) {
    FILE *f = fopen("employes.bin", "rb");
    if (f == NULL) {
        perror("Erreur");
        return 1;
    }
    
    // Lire une structure
    Employe emp;
    if (fread(&emp, sizeof(Employe), 1, f) == 1) {
        printf("Nom: %s, Age: %d, Salaire: %.2f\n", 
               emp.nom, emp.age, emp.salaire);
    }
    
    // Lire un tableau
    int tableau[5];
    if (fread(tableau, sizeof(int), 5, f) == 5) {
        for (int i = 0; i < 5; i++) {
            printf("%d ", tableau[i]);
        }
        printf("\n");
    }
    
    fclose(f);
    return 0;
}
```

### Positionnement dans le fichier

```c
#include <stdio.h>

int main(void) {
    FILE *f = fopen("data.bin", "r+b");
    if (f == NULL) return 1;
    
    // ftell : position actuelle
    long pos = ftell(f);
    printf("Position : %ld\n", pos);
    
    // fseek : déplacement
    fseek(f, 0, SEEK_END);   // Fin du fichier
    long taille = ftell(f);
    printf("Taille : %ld octets\n", taille);
    
    fseek(f, 0, SEEK_SET);   // Début (équivalent à rewind)
    fseek(f, 100, SEEK_CUR); // Avancer de 100 octets
    fseek(f, -50, SEEK_CUR); // Reculer de 50 octets
    
    // rewind : retour au début
    rewind(f);
    
    fclose(f);
    return 0;
}
```

---

## Gestion des erreurs

### Vérification des opérations

```c
#include <stdio.h>
#include <errno.h>
#include <string.h>

int main(void) {
    FILE *f = fopen("inexistant.txt", "r");
    
    if (f == NULL) {
        // Méthode 1 : perror
        perror("Erreur d'ouverture");
        
        // Méthode 2 : errno + strerror
        printf("Erreur %d : %s\n", errno, strerror(errno));
        
        return 1;
    }
    
    // Vérification après lecture/écriture
    char buffer[100];
    if (fgets(buffer, sizeof(buffer), f) == NULL) {
        if (feof(f)) {
            printf("Fin de fichier atteinte\n");
        } else if (ferror(f)) {
            printf("Erreur de lecture\n");
        }
    }
    
    fclose(f);
    return 0;
}
```

### Pattern de gestion d'erreur avec cleanup

```c
#include <stdio.h>
#include <stdlib.h>

int traiter_fichiers(const char *input, const char *output) {
    FILE *fin = NULL;
    FILE *fout = NULL;
    char *buffer = NULL;
    int result = -1;  // Code d'erreur par défaut
    
    fin = fopen(input, "r");
    if (fin == NULL) {
        perror("Erreur input");
        goto cleanup;
    }
    
    fout = fopen(output, "w");
    if (fout == NULL) {
        perror("Erreur output");
        goto cleanup;
    }
    
    buffer = malloc(4096);
    if (buffer == NULL) {
        perror("Erreur malloc");
        goto cleanup;
    }
    
    // Traitement...
    while (fgets(buffer, 4096, fin) != NULL) {
        fputs(buffer, fout);
    }
    
    result = 0;  // Succès
    
cleanup:
    // Nettoyage unique
    if (buffer) free(buffer);
    if (fout) fclose(fout);
    if (fin) fclose(fin);
    
    return result;
}
```

---

## Bonnes pratiques

### Allocation mémoire

```c
// 1. Toujours vérifier le retour de malloc/calloc/realloc
int *p = malloc(n * sizeof(int));
if (p == NULL) {
    // Gérer l'erreur
}

// 2. Utiliser sizeof sur la variable, pas le type
int *arr = malloc(n * sizeof(*arr));  // Plus sûr si le type change

// 3. Mettre à NULL après free
free(p);
p = NULL;

// 4. Vérifier le débordement avant multiplication
if (n > SIZE_MAX / sizeof(int)) {
    // Débordement potentiel!
}

// 5. Libérer dans l'ordre inverse de l'allocation
char *a = malloc(100);
char *b = malloc(200);
// ...
free(b);  // Dernier alloué, premier libéré
free(a);
```

### Fichiers

```c
// 1. Toujours vérifier fopen
FILE *f = fopen("data.txt", "r");
if (f == NULL) {
    perror("Erreur");
    return -1;
}

// 2. Toujours fermer les fichiers
fclose(f);

// 3. Utiliser fgets au lieu de fscanf("%s") pour les chaînes
char buffer[256];
fgets(buffer, sizeof(buffer), f);

// 4. Vérifier les retours de fread/fwrite
size_t n = fread(buffer, 1, sizeof(buffer), f);
if (n < sizeof(buffer) && !feof(f)) {
    // Erreur de lecture
}

// 5. Flush avant changement de mode
fprintf(f, "data");
fflush(f);  // Avant de lire
```

---

## Comparaison avec Python/JS/C#/Go

| Aspect            | C              | Python       | JavaScript  | Go              | 
|-------------------|----------------|--------------|-------------|-----------------|
| Allocation        | `malloc/free`  | Automatique  | Automatique | Automatique     |
| Garbage Collector | Non            | Oui          | Oui         | Oui             |
| Fichiers          | `fopen/fclose` | `open/close` | `fs` module | `os.Open/Close` |
| Gestion erreurs   | Codes retour   | Exceptions   | Exceptions  | `error`         |

```python
# Python : gestion automatique avec context manager
with open("data.txt", "r") as f:
    content = f.read()
# f automatiquement fermé
```

```go
// Go : defer pour garantir la fermeture
f, err := os.Open("data.txt")
if err != nil {
    log.Fatal(err)
}
defer f.Close()  // Appelé automatiquement à la fin
```

---

## Exercices du chapitre

### Exercice 9.1 : Tableau dynamique (Débutant)
Créez des fonctions pour un tableau dynamique d'entiers :
- `int *creer(int capacite)`
- `void detruire(int *arr)`
- `void afficher(int *arr, int taille)`

### Exercice 9.2 : Lecture de fichier (Débutant)
Écrivez un programme qui compte les lignes, mots et caractères d'un fichier (comme `wc`).

### Exercice 9.3 : Copie de fichier (Intermédiaire)
Implémentez une copie de fichier avec :
- Gestion des erreurs
- Barre de progression
- Mode texte et binaire

### Exercice 9.4 : Base de données simple (Intermédiaire)
Créez un système de stockage de contacts :
- Structure `Contact`
- Sauvegarde/chargement binaire
- Ajout, recherche, suppression

### Exercice 9.5 : Tableau redimensionnable (Avancé)
Implémentez un `Vector` dynamique qui double sa capacité automatiquement :
- `Vector *vector_create(void)`
- `void vector_push(Vector *v, int val)`
- `int vector_get(Vector *v, int index)`
- `void vector_destroy(Vector *v)`

---

## Résumé du chapitre

**Segments mémoire** : Stack (automatique), Heap (malloc), Data/BSS (global)

**malloc/calloc/realloc/free** : Allocation dynamique manuelle

**Erreurs courantes** : Fuites, use-after-free, double free, overflow

**Outils** : Valgrind, AddressSanitizer pour détecter les erreurs

**Fichiers** : fopen, fclose, fread, fwrite, fgets, fprintf

**Modes** : `"r"`, `"w"`, `"a"` pour texte, `"rb"`, `"wb"` pour binaire

**Gestion d'erreurs** : Vérifier tous les retours, utiliser goto pour cleanup

---

**Prochain chapitre** : C en cybersécurité et DevSecOps
