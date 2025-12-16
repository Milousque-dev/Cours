# Chapitre 1 : Introduction au langage C et environnement de développement

## Table des matières

1. [Histoire et importance du C](#histoire-et-importance-du-c)
2. [Pourquoi apprendre le C en 2025](#pourquoi-apprendre-le-c-en-2025)
3. [Caractéristiques fondamentales du C](#caractéristiques-fondamentales-du-c)
4. [Comparaison avec les langages modernes](#comparaison-avec-les-langages-modernes)
5. [Installation de l'environnement de développement](#installation-de-lenvironnement-de-développement)
6. [Anatomie d'un programme C](#anatomie-dun-programme-c)
7. [Processus de compilation](#processus-de-compilation)
8. [Premiers pas : Hello World décortiqué](#premiers-pas--hello-world-décortiqué)
9. [Exercices du chapitre](#exercices-du-chapitre)

---

## Histoire et importance du C

### Origines (1969-1973)

Le langage C a été créé entre 1969 et 1973 par **Dennis Ritchie** aux laboratoires Bell (AT&T), en collaboration avec **Ken Thompson**. Il a été développé initialement pour réécrire le système d'exploitation **UNIX**, qui était auparavant écrit en assembleur.

Le C tire son nom de son prédécesseur, le langage **B** (lui-même dérivé de BCPL). La première version stable est apparue en 1972, et le langage a été documenté officiellement dans le livre de référence **"The C Programming Language"** (1978) de Brian Kernighan et Dennis Ritchie, souvent appelé le "K&R".

### Évolution des standards

| Année | Standard          | Nom commun | Principales nouveautés                             |
|-------|-------------------|------------|----------------------------------------------------|
| 1978  |       ---         | K&R C      | Version originale                                  |
| 1989  | ANSI X3.159-1989  | C89/ANSI C | Premier standard officiel                          |
| 1990  | ISO/IEC 9899:1990 | C90        | Adoption ISO du C89                                |
| 1999  | ISO/IEC 9899:1999 | C99        | Variables inline, commentaires //, types long long |
| 2011  | ISO/IEC 9899:2011 | C11        | Multi-threading, _Generic, assertions statiques    |
| 2018  | ISO/IEC 9899:2018 | C17/C18    | Corrections de bugs du C11                         |
| 2023  | ISO/IEC 9899:2023 | C23        | nullptr, attributs, améliorations syntaxiques      |

### Impact mondial du C

Le C a profondément influencé l'informatique moderne :

- **Systèmes d'exploitation** : Linux, Windows, macOS, iOS, Android (kernel)
- **Langages dérivés** : C++, C#, Objective-C, Java, JavaScript, Go, Rust, PHP, Perl
- **Logiciels critiques** : Git, PostgreSQL, SQLite, Redis, Nginx, Apache, Python (interpréteur)
- **Systèmes embarqués** : Microcontrôleurs, automobile, aérospatial, médical

---

## Pourquoi apprendre le C en 2025

### 1. Comprendre le fonctionnement des ordinateurs

Le C est souvent qualifié d'**"assembleur portable"**. Il offre un contrôle direct sur :

- La **mémoire** (allocation, libération, adressage)
- Le **matériel** (registres, ports, interruptions)
- Les **performances** (optimisation fine)

```c
// En C, vous voyez exactement ce qui se passe en mémoire
int x = 42;           // Alloue 4 octets sur la pile
int *ptr = &x;        // Stocke l'adresse de x (ex: 0x7ffd1234)
*ptr = 100;           // Modifie directement cette zone mémoire
```

Comparez avec Python où tout est abstrait :

```python
# Python cache la gestion mémoire
x = 42  # Objet créé quelque part, référencé par x
```

### 2. Performance inégalée

Le C compile en **code machine natif**, sans machine virtuelle ni interpréteur :

| Langage         | Type d'exécution   | Performance relative |
|-----------------|--------------------|----------------------|
| C               | Compilation native | 1x (référence)       |
| C++             | Compilation native | ~1x                  |
| Go              | Compilation native | ~1.5x plus lent      |
| Java            | JVM + JIT          | ~2-3x plus lent      |
| C#              | CLR + JIT          | ~2-3x plus lent      |
| JavaScript (V8) | JIT                | ~3-10x plus lent     |
| Python          | Interprété         | ~50-100x plus lent   |

### 3. Omniprésence dans l'industrie

- **Systèmes embarqués** : 80%+ du code est en C
- **Kernels** : Linux (~27 millions de lignes de C)
- **Bases de données** : MySQL, PostgreSQL, SQLite
- **Langages** : Python, Ruby, PHP sont écrits en C

### 4. Fondation pour la cybersécurité

La majorité des vulnérabilités exploitées concernent du code C/C++ :

- Buffer overflows
- Use-after-free
- Format string attacks
- Integer overflows

Comprendre le C est **indispensable** pour :

- L'analyse de malware
- Le reverse engineering
- Le pentesting bas niveau
- Le développement d'exploits

### 5. Passerelle vers d'autres langages

Une fois le C maîtrisé, vous comprendrez :

- **C++** : Superset du C avec OOP
- **Rust** : Alternative moderne avec sécurité mémoire
- **Go** : Syntaxe inspirée du C, garbage collector
- Les FFI (Foreign Function Interface) de tous les langages

---

## Caractéristiques fondamentales du C

### Langage compilé

Le code source C est transformé en code machine **avant** l'exécution :

```
code.c → Préprocesseur → code.i → Compilateur → code.s → Assembleur → code.o → Éditeur de liens → executable
```

### Typage statique et faible

- **Statique** : Les types sont vérifiés à la compilation
- **Faible** : Conversions implicites possibles (peut être dangereux)

```c
int x = 65;
char c = x;      // Conversion implicite int → char ('A')
float f = x;     // Conversion implicite int → float (65.0)
int *p = x;      // Warning mais souvent compilé (DANGEREUX!)
```

### Paradigme procédural

Le C est un langage **procédural** (pas orienté objet) :

- Organisation en **fonctions**
- Pas de classes ni d'objets natifs
- Pas d'héritage ni de polymorphisme
- Les **structures** regroupent les données

```c
// Pas de classes, mais des structures + fonctions
struct Rectangle {
    int largeur;
    int hauteur;
};

int calculerAire(struct Rectangle r) {
    return r.largeur * r.hauteur;
}
```

### Gestion manuelle de la mémoire

C'est LA différence majeure avec les langages modernes :

| Aspect         | C                          | Python/JS/C#/Go        |
|----------------|----------------------------|------------------------|
| Allocation     | `malloc()` manuel          | Automatique            |
| Libération     | `free()` obligatoire       | Garbage collector      |
| Fuites mémoire | Responsabilité développeur | Impossibles (ou rares) |
| Performances   | Prévisibles                | Pauses GC possibles    |

### Accès bas niveau

Le C permet de manipuler directement :

```c
// Accès direct à une adresse mémoire (registre matériel)
volatile unsigned int *GPIO = (unsigned int *)0x3F200000;
*GPIO = 0x01;  // Écriture directe dans le registre

// Opérations bit à bit
unsigned char flags = 0b00001111;
flags |= (1 << 4);   // Met le bit 4 à 1
flags &= ~(1 << 2);  // Met le bit 2 à 0
```

---

## Comparaison avec les langages modernes

### C vs Python

| Aspect   | C                          | Python                         |
|----------|----------------------------|--------------------------------|
| Typage   | Statique, déclaré          | Dynamique                      |
| Syntaxe  | Accolades, points-virgules | Indentation                    |
| Chaînes  | `char[]` terminé par `\0`  | Type `str` natif               |
| Tableaux | Taille fixe, homogène      | Listes dynamiques, hétérogènes |
| Mémoire  | Manuelle                   | Automatique (GC)               |
| Erreurs  | Comportement indéfini      | Exceptions                     |

```c
// C : déclaration explicite, compilation
int nombres[5] = {1, 2, 3, 4, 5};
for (int i = 0; i < 5; i++) {
    printf("%d\n", nombres[i]);
}
```

```python
# Python : dynamique, interprété
nombres = [1, 2, 3, 4, 5]
for n in nombres:
    print(n)
```

### C vs JavaScript

| Aspect    | C                            | JavaScript                       |
|-----------|------------------------------|----------------------------------|
| Exécution | Compilation native           | Interprété/JIT (navigateur/Node) |
| Types     | `int`, `float`, `char`, etc. | `number`, `string`, `object`     |
| Pointeurs | Oui (fondamentaux)           | Non                              |
| Null      | `NULL` (pointeur)            | `null` et `undefined`            |
| Objets    | Structures                   | Objets prototypes                |

```c
// C : types stricts
int age = 25;
// age = "vingt-cinq";  // ERREUR de compilation
```

```javascript
// JavaScript : types dynamiques
let age = 25;
age = "vingt-cinq";  // Autorisé
```

### C vs C#

| Aspect     | C                          | C#                      |
|------------|----------------------------|-------------------------|
| Paradigme  | Procédural                 | Orienté objet           |
| Mémoire    | Manuelle (`malloc`/`free`) | Automatique (CLR + GC)  |
| Pointeurs  | Partout                    | Mode `unsafe` seulement |
| Chaînes    | `char[]`                   | `string` immuable       |  
| Exceptions | Codes de retour            | `try`/`catch`/`finally` |

```c
// C : allocation manuelle
char *nom = (char *)malloc(50 * sizeof(char));
strcpy(nom, "Alice");
// ... utilisation ...
free(nom);  // OBLIGATOIRE
```

```csharp
// C# : gestion automatique
string nom = "Alice";
// Pas besoin de libérer, le GC s'en charge
```

### C vs Go

Go est souvent présenté comme un "C moderne" :

| Aspect      | C                          | Go                 |
|-------------|----------------------------|--------------------|
| Compilation | Lente (avec optimisations) | Très rapide        |
| Mémoire     | Manuelle                   | Garbage collector  |
| Pointeurs   | Arithmétique autorisée     | Pas d'arithmétique |
| Concurrence | Threads POSIX              | Goroutines natives |
| Erreurs     | Codes de retour            | Valeurs `error`    |

```c
// C : pointeurs avec arithmétique
int arr[] = {10, 20, 30};
int *p = arr;
p++;        // Pointe maintenant sur arr[1]
printf("%d", *p);  // 20
```

```go
// Go : pointeurs sans arithmétique
arr := []int{10, 20, 30}
p := &arr[0]
// p++  // ERREUR : arithmétique interdite
fmt.Println(*p)  // 10
```

---

## Installation de l'environnement de développement

### Compilateurs disponibles

#### GCC (GNU Compiler Collection)

Le compilateur standard sur Linux/Unix :

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install build-essential

# Vérification
gcc --version
```

#### Clang (LLVM)

Compilateur moderne avec meilleurs messages d'erreur :

```bash
# Ubuntu/Debian
sudo apt install clang

# Vérification
clang --version
```

#### MinGW (Windows)

Pour Windows sans Visual Studio :

1. Télécharger [MinGW-w64](https://www.mingw-w64.org/)
2. Ajouter `bin/` au PATH
3. Utiliser `gcc` ou `mingw32-gcc`

#### MSVC (Microsoft Visual C++)

Inclus avec Visual Studio sur Windows :

1. Installer Visual Studio (Community gratuit)
2. Sélectionner "Développement Desktop C++"
3. Utiliser le terminal "Developer Command Prompt"

### Éditeurs et IDE recommandés

| Outil             | Type    | Plateformes | Points forts            |
|-------------------|---------|-------------|-------------------------|
| **VS Code**       | Éditeur | Tous        | Extensions C/C++, léger |
| **CLion**         | IDE     | Tous        | Debugger intégré, CMake |
| **Visual Studio** | IDE     | Windows     | Complet, lourd          |
| **Code::Blocks**  | IDE     | Tous        | Simple, léger           |
| **Vim/Neovim**    | Éditeur | Tous        | Rapide, configurable    |

### Configuration VS Code pour C

1. Installer VS Code
2. Installer l'extension **"C/C++"** de Microsoft
3. Installer l'extension **"Code Runner"** (optionnel)

Créer `.vscode/tasks.json` :

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Compiler C",
            "type": "shell",
            "command": "gcc",
            "args": [
                "-g",
                "-Wall",
                "-Wextra",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

### Options de compilation essentielles

```bash
# Options recommandées pour le développement
gcc -Wall -Wextra -Werror -std=c11 -g programme.c -o programme

# Explication :
# -Wall      : Active la plupart des warnings
# -Wextra    : Warnings supplémentaires
# -Werror    : Traite les warnings comme des erreurs
# -std=c11   : Utilise le standard C11
# -g         : Inclut les infos de debug
# -o nom     : Nom du fichier de sortie
```

Options pour la production :

```bash
# Optimisations pour la production
gcc -O2 -DNDEBUG programme.c -o programme

# -O2        : Optimisation niveau 2 (bon équilibre)
# -O3        : Optimisation maximale
# -Os        : Optimise pour la taille
# -DNDEBUG   : Désactive les assertions
```

---

## Anatomie d'un programme C

### Structure générale

```c
/* ============================================
   Structure type d'un programme C
   ============================================ */

// 1. DIRECTIVES PRÉPROCESSEUR
#include <stdio.h>      // Bibliothèque standard I/O
#include <stdlib.h>     // Fonctions utilitaires (malloc, exit...)
#include <string.h>     // Manipulation de chaînes
#include "mon_header.h" // Header personnel (guillemets)

// 2. MACROS ET CONSTANTES
#define PI 3.14159
#define MAX(a, b) ((a) > (b) ? (a) : (b))

// 3. DÉCLARATIONS DE TYPES
typedef struct {
    char nom[50];
    int age;
} Personne;

// 4. PROTOTYPES DE FONCTIONS (déclarations)
void afficherMessage(const char *msg);
int calculerSomme(int a, int b);

// 5. VARIABLES GLOBALES (à éviter si possible)
int compteurGlobal = 0;

// 6. FONCTION PRINCIPALE
int main(int argc, char *argv[]) {
    // Point d'entrée du programme
    printf("Arguments reçus : %d\n", argc);
    
    afficherMessage("Bonjour !");
    
    int resultat = calculerSomme(5, 3);
    printf("5 + 3 = %d\n", resultat);
    
    return 0;  // Code de sortie (0 = succès)
}

// 7. DÉFINITIONS DES FONCTIONS
void afficherMessage(const char *msg) {
    printf("%s\n", msg);
}

int calculerSomme(int a, int b) {
    return a + b;
}
```

### Les directives `#include`

Il existe deux types d'inclusions :

```c
#include <stdio.h>      // Cherche dans les chemins système (/usr/include...)
#include "monfichier.h" // Cherche d'abord dans le répertoire courant
```

Headers standards les plus utilisés :

| Header        | Contenu                                          |
|---------------|--------------------------------------------------|
| `<stdio.h>`   | Entrées/sorties (printf, scanf, FILE...)         |
| `<stdlib.h>`  | Utilitaires (malloc, free, exit, atoi...)        |
| `<string.h>`  | Chaînes (strlen, strcpy, strcmp...)              |
| `<math.h>`    | Mathématiques (sin, cos, sqrt, pow...)           |
| `<ctype.h>`   | Caractères (isalpha, isdigit, toupper...)        |
| `<time.h>`    | Date et heure (time, clock...)                   |
| `<stdbool.h>` | Booléens (bool, true, false) - C99+              |
| `<stdint.h>`  | Types entiers fixes (int32_t, uint8_t...) - C99+ |

### La fonction `main()`

Le point d'entrée obligatoire de tout programme C :

```c
// Forme simple
int main(void) {
    // ...
    return 0;
}

// Forme avec arguments de ligne de commande
int main(int argc, char *argv[]) {
    // argc : nombre d'arguments (au moins 1 : le nom du programme)
    // argv : tableau de chaînes (les arguments)
    
    printf("Programme : %s\n", argv[0]);
    
    for (int i = 1; i < argc; i++) {
        printf("Argument %d : %s\n", i, argv[i]);
    }
    
    return 0;
}

// Équivalent avec pointeur de pointeur
int main(int argc, char **argv) {
    // char **argv est équivalent à char *argv[]
    return 0;
}
```

### Valeurs de retour de `main()`

```c
return 0;              // Succès (convention)
return EXIT_SUCCESS;   // Succès (macro de stdlib.h)
return EXIT_FAILURE;   // Échec (macro de stdlib.h)
return 1;              // Échec (convention)
```

Le code de retour peut être récupéré par le shell :

```bash
./programme
echo $?   # Affiche le code de retour (0-255)
```

---

## Processus de compilation

### Les 4 étapes de compilation

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROCESSUS DE COMPILATION                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  programme.c                                                    │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────┐                                            │
│  │ 1.PRÉPROCESSEUR │  Traite #include, #define, #ifdef          │
│  │   (cpp)         │  Expansion des macros                      │
│  └────────┬────────┘                                            │
│           ▼                                                     │
│  programme.i (code prétraité)                                   │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                            │
│  │ 2.COMPILATEUR   │  Analyse syntaxique et sémantique          │
│  │   (cc1)         │  Génère le code assembleur                 │
│  └────────┬────────┘                                            │
│           ▼                                                     │
│  programme.s (code assembleur)                                  │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                            │
│  │ 3.ASSEMBLEUR    │  Traduit en code machine                   │
│  │   (as)          │  Crée le fichier objet                     │
│  └────────┬────────┘                                            │
│           ▼                                                     │
│  programme.o (fichier objet)                                    │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                            │
│  │ 4.ÉDITEUR DE    │  Lie avec les bibliothèques                │
│  │   LIENS (ld)    │  Résout les symboles                       │
│  └────────┬────────┘                                            │
│           ▼                                                     │
│  programme (exécutable)                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Observer chaque étape

```bash
# Étape 1 : Préprocesseur seulement
gcc -E programme.c -o programme.i

# Étape 2 : Compilation en assembleur
gcc -S programme.c -o programme.s

# Étape 3 : Assemblage en objet
gcc -c programme.c -o programme.o

# Étape 4 : Édition de liens (implicite avec gcc)
gcc programme.o -o programme
```

### Exemple concret

Fichier `test.c` :

```c
#include <stdio.h>

#define MESSAGE "Bonjour"

int main(void) {
    printf("%s\n", MESSAGE);
    return 0;
}
```

Après préprocesseur (`gcc -E test.c`) :

```c
// ... des milliers de lignes de stdio.h ...

int main(void) {
    printf("%s\n", "Bonjour");  // MESSAGE remplacé
    return 0;
}
```

### Compilation séparée (projets multi-fichiers)

```bash
# Compiler chaque fichier en objet
gcc -c main.c -o main.o
gcc -c utils.c -o utils.o
gcc -c calculs.c -o calculs.o

# Lier tous les objets
gcc main.o utils.o calculs.o -o programme
```

Avec un Makefile :

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -std=c11

programme: main.o utils.o calculs.o
	$(CC) $^ -o $@

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f *.o programme
```

---

## Premiers pas : Hello World décortiqué

### Le programme minimal

```c
#include <stdio.h>

int main(void) {
    printf("Hello, World!\n");
    return 0;
}
```

Analysons chaque élément :

#### `#include <stdio.h>`

- **#** : Indique une directive préprocesseur
- **include** : Instruction d'inclusion
- **<stdio.h>** : "Standard Input/Output Header"
- Contient la déclaration de `printf()`, `scanf()`, etc.

#### `int main(void)`

- **int** : Type de retour (entier)
- **main** : Nom de la fonction principale (obligatoire)
- **(void)** : Pas de paramètres (C recommande `void` plutôt que vide)

#### `printf("Hello, World!\n");`

- **printf** : "Print Formatted" - affiche du texte formaté
- **"..."** : Chaîne de caractères (guillemets doubles)
- **\n** : Caractère d'échappement "nouvelle ligne"
- **;** : Fin d'instruction obligatoire

#### `return 0;`

- Renvoie 0 au système d'exploitation
- 0 signifie "exécution réussie"

### Séquences d'échappement

| Séquence | Signification  | Code ASCII |
|----------|----------------|------------|
| `\n`     | Nouvelle ligne | 10         |
| `\t`     | Tabulation     | 9          |
| `\r`     | Retour chariot | 13         |
| `\\`     | Backslash      | 92         |
| `\'`     | Apostrophe     | 39         |
| `\"`     | Guillemet      | 34         |
| `\0`     | Caractère nul  | 0          |
| `\a`     | Bip sonore     | 7          |
| `\b`     | Retour arrière | 8          |

### Compilation et exécution

```bash
# Compilation
gcc -Wall hello.c -o hello

# Exécution
./hello
Hello, World!

# Vérifier le code de retour
echo $?
0
```

### Variantes et expérimentations

```c
#include <stdio.h>

int main(void) {
    // Plusieurs printf
    printf("Ligne 1\n");
    printf("Ligne 2\n");
    
    // Un seul printf multi-lignes
    printf("Ligne 3\n"
           "Ligne 4\n"
           "Ligne 5\n");
    
    // Caractères spéciaux
    printf("Tabulation:\tici\n");
    printf("Citation: \"Hello\"\n");
    printf("Chemin: C:\\Users\\Nom\n");
    
    // puts() : alternative simple (ajoute \n automatiquement)
    puts("Avec puts");
    
    // putchar() : un seul caractère
    putchar('X');
    putchar('\n');
    
    return 0;
}
```

---

## Exercices du chapitre

### Exercice 1.1 : Installation et premier programme (Débutant)

**Objectif** : Vérifier que l'environnement est fonctionnel.

1. Installez GCC ou Clang sur votre système
2. Créez un fichier `test.c` avec un Hello World
3. Compilez avec `gcc -Wall test.c -o test`
4. Exécutez et vérifiez le code de retour

**Attendu** :
```
Hello, World!
(code de retour : 0)
```

---

### Exercice 1.2 : Message personnalisé (Débutant)

**Objectif** : Manipuler `printf` et les séquences d'échappement.

Créez un programme qui affiche exactement :

```
=====================================
|   Bienvenue dans le cours de C   |
|        Par [Votre Nom]           |
=====================================
	-> Version 1.0
	-> Date: 2025
```

**Indice** : Utilisez `\t` pour les tabulations et calculez les espaces pour l'alignement.

---

### Exercice 1.3 : Art ASCII (Débutant)

**Objectif** : Maîtriser l'affichage multi-lignes.

Affichez ce dessin :

```
    /\
   /  \
  /    \
 /______\
    ||
    ||
   /||\
```

---

### Exercice 1.4 : Arguments de ligne de commande (Intermédiaire)

**Objectif** : Comprendre `argc` et `argv`.

Créez un programme qui :

1. Affiche le nombre d'arguments reçus
2. Affiche chaque argument avec son numéro
3. Retourne `1` si aucun argument n'est fourni (hors nom du programme)

**Exemple** :
```bash
./programme hello world 42
Arguments : 4
[0] : ./programme
[1] : hello
[2] : world
[3] : 42
```

---

### Exercice 1.5 : Exploration du préprocesseur (Intermédiaire)

**Objectif** : Observer le travail du préprocesseur.

1. Créez ce fichier :

```c
#include <stdio.h>

#define ANNEE 2025
#define SALUER(nom) printf("Bonjour, " nom "!\n")

int main(void) {
    printf("Nous sommes en %d\n", ANNEE);
    SALUER("Alice");
    return 0;
}
```

2. Exécutez `gcc -E fichier.c | tail -20` pour voir le résultat du préprocesseur
3. Expliquez ce que vous observez

---

### Exercice 1.6 : Codes de retour (Intermédiaire)

**Objectif** : Comprendre les codes de sortie.

Créez un programme qui :

1. Prend un argument numérique
2. Retourne ce nombre comme code de sortie
3. Affiche un message différent selon que le nombre est 0 ou non

**Test** :
```bash
./programme 0
echo $?  # Doit afficher 0

./programme 42
echo $?  # Doit afficher 42
```

**Indice** : Utilisez `atoi()` de `<stdlib.h>` pour convertir une chaîne en entier.

---

### Exercice 1.7 : Comparaison C/Python (Réflexion)

**Objectif** : Comprendre les différences fondamentales.

Vous avez ce code Python :

```python
nom = "Alice"
age = 25
taille = 1.65

print(f"{nom} a {age} ans et mesure {taille}m")
print(type(nom), type(age), type(taille))

# Dynamisme
age = "vingt-cinq"  # Possible en Python
```

Questions :

1. Quel serait l'équivalent en C (sans se soucier de la syntaxe exacte) ?
2. Pourquoi `age = "vingt-cinq"` serait impossible en C ?
3. Quel avantage et inconvénient cela représente ?

---

### Exercice 1.8 : Makefile basique (Intermédiaire-Avancé)

**Objectif** : Automatiser la compilation.

Créez un projet avec :

- `main.c` : fonction main qui appelle `afficher_info()`
- `info.c` : définit `afficher_info()` qui affiche "Module info"
- `info.h` : déclare `afficher_info()`
- `Makefile` : compile le projet avec `make`

**Structure** :
```
projet/
├── main.c
├── info.c
├── info.h
└── Makefile
```

---

## Résumé du chapitre

✅ Le C est un langage **compilé, typé statiquement, procédural**

✅ Créé en 1972, il reste **fondamental** en 2025 (systèmes, embarqué, sécurité)

✅ Différences clés avec Python/JS/C#/Go :
- Gestion mémoire **manuelle**
- Typage **strict**
- **Pointeurs** natifs
- Pas de garbage collector

✅ Processus de compilation : Préprocesseur → Compilateur → Assembleur → Éditeur de liens

✅ Programme minimal : `#include`, `main()`, `printf()`, `return`

✅ Options de compilation essentielles : `-Wall -Wextra -std=c11 -g`

---

## Pour aller plus loin

- **Livre** : "The C Programming Language" (K&R) - La référence historique
- **Standard** : Lire les sections du standard C11/C17
- **Pratique** : [Exercism C Track](https://exercism.org/tracks/c)
- **Documentation** : [cppreference.com](https://en.cppreference.com/w/c)

---

**Prochain chapitre** : Variables, types de données et constantes
