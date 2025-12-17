# Chapitre 10 : C en cybersécurité et DevSecOps

## Table des matières

1.  [Pourquoi le C en cybersécurité](#pourquoi-le-c-en-cybersécurité)
2.  [Vulnérabilités classiques du C](#vulnérabilités-classiques-du-c)
3.  [Buffer Overflow](#buffer-overflow)
4.  [Format String Attack](#format-string-attack)
5.  [Integer Overflow](#integer-overflow)
6.  [Use-After-Free et Double Free](#use-after-free-et-double-free)
7.  [Fonctions dangereuses vs sécurisées](#fonctions-dangereuses-vs-sécurisées)
8.  [Mécanismes de protection](#mécanismes-de-protection)
9.  [Outils d'analyse statique](#outils-danalyse-statique)
10. [Outils d'analyse dynamique](#outils-danalyse-dynamique)
11. [Intégration DevSecOps et CI/CD](#intégration-devsecops-et-cicd)
12. [Bonnes pratiques de développement sécurisé](#bonnes-pratiques-de-développement-sécurisé)
13. [Exercices du chapitre](#exercices-du-chapitre)

---

## Pourquoi le C en cybersécurité

### Le C : épée à double tranchant

Le C est omniprésent dans les systèmes critiques, ce qui en fait une cible privilégiée :

| Domaine                 | Exemples                     | Risques                      |
|-------------------------|------------------------------|------------------------------|
| Systèmes d'exploitation | Linux, Windows, macOS        | Élévation de privilèges      |
| Navigateurs             | Chrome, Firefox (parties)    | RCE, sandbox escape          |
| Bases de données        | MySQL, PostgreSQL, SQLite    | Corruption, fuite de données |
| Serveurs web            | Nginx, Apache                | RCE, DoS                     |
| IoT/Embarqué            | Routeurs, caméras, véhicules | Prise de contrôle à distance |

### Statistiques de vulnérabilités

Selon le CWE (Common Weakness Enumeration), les vulnérabilités les plus courantes en C :

1. **CWE-120** : Buffer Copy without Checking Size (Buffer Overflow)
2. **CWE-416** : Use After Free
3. **CWE-190** : Integer Overflow
4. **CWE-134** : Uncontrolled Format String
5. **CWE-787** : Out-of-bounds Write
6. **CWE-125** : Out-of-bounds Read

### Pourquoi tant de vulnérabilités ?

```c
// Le C fait confiance au développeur pour TOUT vérifier
char buffer[10];
strcpy(buffer, user_input);  // Aucune vérification de taille!

// Pas de vérification des bornes
int arr[5];
arr[100] = 42;  // Compile sans erreur!

// Gestion mémoire manuelle
int *p = malloc(100);
free(p);
*p = 42;  // Use-after-free : compile sans erreur!
```

---

## Vulnérabilités classiques du C

### Vue d'ensemble

```
Vulnérabilités C
├── Mémoire
│   ├── Buffer Overflow (stack/heap)
│   ├── Use-After-Free
│   ├── Double Free
│   ├── Memory Leak
│   └── Uninitialized Memory
│
├── Entiers
│   ├── Integer Overflow/Underflow
│   ├── Sign Confusion
│   └── Truncation
│
├── Format
│   └── Format String Attack
│
└── Logique
    ├── Race Conditions
    ├── Time-of-check/Time-of-use (TOCTOU)
    └── NULL Pointer Dereference
```

---

## Buffer Overflow

### Stack Buffer Overflow

Le plus classique des exploits :

```c
#include <stdio.h>
#include <string.h>

void fonction_vulnerable(char *input) {
    char buffer[64];
    strcpy(buffer, input);  // VULNÉRABLE : pas de vérification de taille
    printf("Vous avez entré : %s\n", buffer);
}

int main(int argc, char *argv[]) {
    if (argc > 1) {
        fonction_vulnerable(argv[1]);
    }
    return 0;
}
```

**Exploitation :**
```
Stack avant overflow:
┌──────────────────────┐
│  Return Address      │ ← Cible de l'attaquant
├──────────────────────┤
│  Saved EBP           │
├──────────────────────┤
│  buffer[63]          │
│  ...                 │
│  buffer[0]           │ ← strcpy commence ici
└──────────────────────┘

Stack après overflow:
┌──────────────────────┐
│  0x41414141 (AAAA)   │ ← Return address écrasée!
├──────────────────────┤
│  0x41414141          │
├──────────────────────┤
│  AAAAAAAAAAAAAAAA    │
│  AAAAAAAAAAAAAAAA    │
│  AAAAAAAAAAAAAAAA    │
└──────────────────────┘
```

### Heap Buffer Overflow

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void) {
    char *buffer1 = malloc(32);
    char *buffer2 = malloc(32);
    
    // Metadata du heap entre buffer1 et buffer2
    // Un overflow peut corrompre ces métadonnées
    
    strcpy(buffer1, "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA");
    // Écrit au-delà de buffer1, corrompt les métadonnées ou buffer2
    
    free(buffer1);  // Peut crasher ou permettre l'exécution de code
    free(buffer2);
    
    return 0;
}
```

### Correction sécurisée

```c
#include <stdio.h>
#include <string.h>

void fonction_securisee(const char *input) {
    char buffer[64];
    
    // Méthode 1 : strncpy + terminaison manuelle
    strncpy(buffer, input, sizeof(buffer) - 1);
    buffer[sizeof(buffer) - 1] = '\0';
    
    // Méthode 2 : snprintf (recommandé)
    snprintf(buffer, sizeof(buffer), "%s", input);
    
    // Méthode 3 : vérification préalable
    if (strlen(input) >= sizeof(buffer)) {
        fprintf(stderr, "Entrée trop longue!\n");
        return;
    }
    strcpy(buffer, input);  // Maintenant sûr
    
    printf("Vous avez entré : %s\n", buffer);
}
```

---

## Format String Attack

### Vulnérabilité

```c
#include <stdio.h>

void vulnerable(char *user_input) {
    printf(user_input);  // VULNÉRABLE!
    // L'attaquant contrôle la format string
}

void securise(char *user_input) {
    printf("%s", user_input);  // SÉCURISÉ
    // La format string est fixe
}
```

### Exploitation

```c
// Si l'attaquant entre : "%x %x %x %x"
// printf lit des valeurs sur la stack!

// Si l'attaquant entre : "%s%s%s%s"
// printf peut crasher en déréférençant des pointeurs invalides

// Si l'attaquant entre : "%n"
// printf ÉCRIT en mémoire (nombre de caractères affichés)
```

**Exemple d'exploitation :**
```bash
# Lecture de la stack
./programme "%08x.%08x.%08x.%08x"
# Affiche : deadbeef.cafebabe.12345678.90abcdef

# Lecture arbitraire de mémoire
./programme "AAAA%7$s"
# Peut lire la chaîne à l'adresse 0x41414141

# Écriture arbitraire (le plus dangereux)
./programme $(python -c 'print "\x10\x20\x30\x40" + "%08x" * 5 + "%n"')
# Écrit une valeur à l'adresse 0x40302010
```

### Code vulnérable typique

```c
#include <stdio.h>
#include <syslog.h>

void log_erreur(char *message) {
    // VULNÉRABLE : le message contrôle la format string
    syslog(LOG_ERR, message);
    fprintf(stderr, message);
    printf(message);
}

void log_erreur_securise(char *message) {
    // SÉCURISÉ : format string fixe
    syslog(LOG_ERR, "%s", message);
    fprintf(stderr, "%s", message);
    printf("%s", message);
}
```

---

## Integer Overflow

### Dépassement d'entier

```c
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>

void vulnerable(size_t nmemb, size_t size) {
    // VULNÉRABLE : multiplication peut déborder
    size_t total = nmemb * size;  // Si nmemb * size > SIZE_MAX, wraparound!
    
    char *buffer = malloc(total);  // Alloue moins que prévu
    // ... écriture de nmemb * size octets → overflow!
}

void securise(size_t nmemb, size_t size) {
    // Vérification avant multiplication
    if (nmemb != 0 && size > SIZE_MAX / nmemb) {
        fprintf(stderr, "Integer overflow détecté!\n");
        return;
    }
    
    size_t total = nmemb * size;
    char *buffer = malloc(total);
    // ...
}
```

### Confusion de signe

```c
#include <stdio.h>
#include <string.h>

// VULNÉRABLE : confusion signed/unsigned
void copier_donnees(char *dest, char *src, int longueur) {
    if (longueur > 100) {  // Vérification de sécurité
        printf("Trop long!\n");
        return;
    }
    // Si longueur = -1 (signé), devient ~4 milliards en unsigned!
    memcpy(dest, src, longueur);  // memcpy prend size_t (unsigned)
}

// SÉCURISÉ : utiliser le bon type
void copier_donnees_secure(char *dest, char *src, size_t longueur) {
    if (longueur > 100) {
        printf("Trop long!\n");
        return;
    }
    memcpy(dest, src, longueur);
}
```

### Troncature

```c
#include <stdio.h>

void vulnerable(void) {
    size_t grande_valeur = 0x100000001;  // Plus grand que 32 bits
    
    // Troncature si int est 32 bits
    int petite_valeur = (int)grande_valeur;  // petite_valeur = 1 !
    
    char *buffer = malloc(petite_valeur);  // Alloue 1 octet
    // ... mais on pense avoir alloué ~4 Go
}
```

---

## Use-After-Free et Double Free

### Use-After-Free (CWE-416)

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char *nom;
    void (*callback)(void);
} Utilisateur;

void afficher(void) {
    printf("Utilisateur actif\n");
}

int main(void) {
    Utilisateur *user = malloc(sizeof(Utilisateur));
    user->nom = strdup("Alice");
    user->callback = afficher;
    
    // Libération
    free(user->nom);
    free(user);
    
    // USE-AFTER-FREE!
    user->callback();  // Appelle une fonction potentiellement contrôlée
    // L'attaquant peut réallouer la mémoire et y placer un pointeur malveillant
    
    return 0;
}
```

### Exploitation

```c
// Scénario d'exploitation
Utilisateur *user = malloc(sizeof(Utilisateur));
user->callback = fonction_legitime;
free(user);

// L'attaquant alloue la même taille
char *malicious = malloc(sizeof(Utilisateur));
// Écrit un pointeur vers du code malveillant
*(void**)(malicious + offsetof(Utilisateur, callback)) = shellcode;

// Le programme utilise l'ancien pointeur
user->callback();  // Exécute le shellcode!
```

### Double Free (CWE-415)

```c
#include <stdlib.h>

int main(void) {
    char *p = malloc(100);
    
    free(p);
    free(p);  // DOUBLE FREE!
    
    // Peut corrompre les métadonnées du heap
    // Peut permettre l'exécution de code arbitraire
    
    return 0;
}
```

### Correction

```c
#include <stdlib.h>

// Pattern sécurisé
void safe_free(void **ptr) {
    if (ptr != NULL && *ptr != NULL) {
        free(*ptr);
        *ptr = NULL;  // Empêche double free et use-after-free
    }
}

int main(void) {
    char *p = malloc(100);
    
    safe_free((void**)&p);  // p = NULL après
    safe_free((void**)&p);  // Ne fait rien (p est NULL)
    
    // Tentative d'utilisation détectée
    if (p != NULL) {
        // ...
    }
    
    return 0;
}
```

---

## Fonctions dangereuses vs sécurisées

### Tableau de correspondance

| Dangereuse    | Problème            | Alternative sécurisée      |
|---------------|---------------------|----------------------------|
| `gets()`      | Aucune limite       | `fgets()`                  |
| `strcpy()`    | Pas de vérification | `strncpy()`, `strlcpy()`   |
| `strcat()`    | Pas de vérification | `strncat()`, `strlcat()`   |
| `sprintf()`   | Pas de limite       | `snprintf()`               |
| `scanf("%s")` | Pas de limite       | `scanf("%Ns")` avec N fixe |
| `vsprintf()`  | Pas de limite       | `vsnprintf()`              |
| `realpath()`  | Buffer overflow     | Spécifier la taille        |
| `getwd()`     | Buffer overflow     | `getcwd()`                 |

### Exemples de correction

```c
#include <stdio.h>
#include <string.h>

void exemples_securises(void) {
    char buffer[100];
    char src[] = "Hello, World!";
    
    // Au lieu de gets() - RETIRÉ du standard!
    // gets(buffer);
    if (fgets(buffer, sizeof(buffer), stdin) != NULL) {
        buffer[strcspn(buffer, "\n")] = '\0';  // Retire le \n
    }
    
    // Au lieu de strcpy()
    // strcpy(buffer, src);
    strncpy(buffer, src, sizeof(buffer) - 1);
    buffer[sizeof(buffer) - 1] = '\0';
    
    // Ou mieux avec snprintf
    snprintf(buffer, sizeof(buffer), "%s", src);
    
    // Au lieu de strcat()
    char dest[100] = "Hello, ";
    // strcat(dest, src);
    size_t remaining = sizeof(dest) - strlen(dest) - 1;
    strncat(dest, src, remaining);
    
    // Au lieu de sprintf()
    // sprintf(buffer, "Valeur: %d", 42);
    snprintf(buffer, sizeof(buffer), "Valeur: %d", 42);
    
    // Au lieu de scanf("%s")
    // scanf("%s", buffer);
    scanf("%99s", buffer);  // Limite à 99 caractères
}
```

### Fonctions interdites (bannies dans de nombreux projets)

```c
// Liste noire typique dans un projet sécurisé :
// gets, strcpy, strcat, sprintf, vsprintf, scanf (sans limite)
// strncpy (difficile à utiliser correctement)
// atoi, atol, atof (pas de gestion d'erreur)

// Préférer :
// fgets, strlcpy/snprintf, strlcat/snprintf, snprintf, vsnprintf
// scanf avec limites, strtol, strtod (avec gestion d'erreur)
```

---

## Mécanismes de protection

### Protection côté système

#### ASLR (Address Space Layout Randomization)

Randomise les adresses de la stack, du heap et des bibliothèques :

```bash
# Vérifier l'état d'ASLR (Linux)
cat /proc/sys/kernel/randomize_va_space
# 0 = désactivé, 1 = partiel, 2 = complet

# Désactiver temporairement (pour debug)
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
```

#### DEP/NX (Data Execution Prevention / No-eXecute)

Empêche l'exécution de code dans la stack/heap :

```bash
# Vérifier si un binaire a NX activé
readelf -l programme | grep GNU_STACK
# RW = NX activé, RWE = NX désactivé
```

#### Stack Canaries (Stack Protector)

Valeur aléatoire entre le buffer et l'adresse de retour :

```c
// Compilation avec protection
gcc -fstack-protector-strong programme.c -o programme

// Fonctionnement :
// 1. Au début de la fonction, un canary est placé sur la stack
// 2. Avant le return, le canary est vérifié
// 3. Si modifié → détection de l'overflow → abort()
```

#### RELRO (RELocation Read-Only)

Protège la GOT (Global Offset Table) contre l'écriture :

```bash
# Compilation avec Full RELRO
gcc -Wl,-z,relro,-z,now programme.c -o programme
```

### Protection côté compilation

```bash
# Compilation sécurisée complète
gcc -Wall -Wextra -Werror \
    -fstack-protector-strong \
    -D_FORTIFY_SOURCE=2 \
    -O2 \
    -Wformat -Wformat-security \
    -fPIE -pie \
    -Wl,-z,relro,-z,now \
    programme.c -o programme
```

| Option                     | Protection                             |
|----------------------------|----------------------------------------|  
| `-fstack-protector-strong` | Stack canaries                         |
| `-D_FORTIFY_SOURCE=2`      | Vérifie strcpy, memcpy, etc.           |
| `-fPIE -pie`               | Position Independent Executable (ASLR) |
| `-Wl,-z,relro,-z,now`      | Full RELRO                             |
| `-Wformat-security`        | Détecte format string vulnérables      |

---

## Outils d'analyse statique

### Cppcheck

Analyse statique gratuite et open-source :

```bash
# Installation
sudo apt install cppcheck

# Utilisation basique
cppcheck --enable=all programme.c

# Avec rapport
cppcheck --enable=all --xml 2> rapport.xml programme.c

# Exemple de sortie
# [programme.c:15]: (error) Buffer overflow: strcpy
# [programme.c:23]: (warning) scanf without width limit
```

### Clang Static Analyzer

Intégré à LLVM/Clang :

```bash
# Analyse simple
scan-build gcc -c programme.c

# Génération de rapport HTML
scan-build -o rapport/ gcc programme.c -o programme

# Options avancées
scan-build -enable-checker security.insecureAPI.gets \
           -enable-checker security.insecureAPI.strcpy \
           gcc programme.c
```

### Flawfinder

Recherche les patterns dangereux :

```bash
# Installation
pip install flawfinder

# Utilisation
flawfinder programme.c

# Avec niveau de risque minimum
flawfinder --minlevel=3 programme.c

# Exemple de sortie
# programme.c:15:  [4] (buffer) strcpy:
#   Does not check for buffer overflows when copying to destination
```

### Coverity (commercial)

Analyse statique avancée, gratuite pour l'open-source :

```bash
# Soumission d'un projet
cov-build --dir cov-int make
tar czvf project.tgz cov-int
# Upload sur scan.coverity.com
```

### Exemple de script d'analyse

```bash
#!/bin/bash
# analyse_securite.sh

echo "=== Analyse de sécurité ==="

echo -e "\n[1/3] Cppcheck..."
cppcheck --enable=all --error-exitcode=1 src/

echo -e "\n[2/3] Flawfinder..."
flawfinder --minlevel=2 src/

echo -e "\n[3/3] Clang Static Analyzer..."
scan-build -o reports/ make clean all

echo -e "\n=== Analyse terminée ==="
```

---

## Outils d'analyse dynamique

### Valgrind

Détection de fuites et erreurs mémoire :

```bash
# Installation
sudo apt install valgrind

# Détection de fuites
valgrind --leak-check=full ./programme

# Détection de use-after-free, double-free
valgrind --track-origins=yes ./programme

# Profiling mémoire
valgrind --tool=massif ./programme
ms_print massif.out.*
```

### AddressSanitizer (ASan)

Détection rapide d'erreurs mémoire :

```bash
# Compilation
gcc -fsanitize=address -g programme.c -o programme

# Exécution (détecte automatiquement)
./programme

# Exemple de sortie ASan
# ==12345==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x...
# READ of size 1 at 0x... thread T0
#     #0 0x... in main programme.c:15
```

### UndefinedBehaviorSanitizer (UBSan)

Détecte les comportements indéfinis :

```bash
# Compilation
gcc -fsanitize=undefined -g programme.c -o programme

# Détecte : integer overflow, division par zéro, null pointer, etc.
```

### MemorySanitizer (MSan)

Détecte les lectures de mémoire non initialisée :

```bash
# Avec Clang uniquement
clang -fsanitize=memory -g programme.c -o programme
```

### Fuzzing avec AFL (American Fuzzy Lop)

Test automatique avec entrées aléatoires :

```bash
# Installation
sudo apt install afl++

# Compilation avec instrumentation
afl-gcc -o programme_fuzz programme.c

# Créer des cas de test initiaux
mkdir testcases
echo "test" > testcases/test1

# Lancer le fuzzing
afl-fuzz -i testcases -o findings ./programme_fuzz @@
```

### libFuzzer (intégré à Clang)

```c
// programme_fuzz.c
#include <stdint.h>
#include <stddef.h>

int LLVMFuzzerTestOneInput(const uint8_t *data, size_t size) {
    // Fonction à tester
    if (size > 0 && data[0] == 'F') {
        if (size > 1 && data[1] == 'U') {
            if (size > 2 && data[2] == 'Z') {
                if (size > 3 && data[3] == 'Z') {
                    __builtin_trap();  // Bug trouvé!
                }
            }
        }
    }
    return 0;
}
```

```bash
# Compilation
clang -fsanitize=fuzzer,address programme_fuzz.c -o fuzzer

# Exécution
./fuzzer
```

---

## Intégration DevSecOps et CI/CD

### Pipeline CI/CD sécurisé

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - security
  - deploy

variables:
  CFLAGS: "-Wall -Wextra -Werror -fstack-protector-strong"

build:
  stage: build
  script:
    - make clean
    - make CFLAGS="$CFLAGS"
  artifacts:
    paths:
      - build/

test:
  stage: test
  script:
    - make test
    - ./run_tests.sh

static_analysis:
  stage: security
  script:
    - cppcheck --enable=all --error-exitcode=1 src/
    - flawfinder --minlevel=3 --error-level=3 src/
  allow_failure: false

dynamic_analysis:
  stage: security
  script:
    - make CFLAGS="$CFLAGS -fsanitize=address,undefined"
    - ./run_tests.sh
    - valgrind --leak-check=full --error-exitcode=1 ./programme

fuzzing:
  stage: security
  script:
    - afl-gcc -o fuzz_target src/*.c
    - timeout 3600 afl-fuzz -i testcases -o findings ./fuzz_target @@
  when: scheduled  # Exécution planifiée, pas à chaque commit

deploy:
  stage: deploy
  script:
    - make install
  only:
    - main
  when: manual
```

### GitHub Actions

```yaml
# .github/workflows/security.yml
name: Security Analysis

on: [push, pull_request]

jobs:
  static-analysis:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install tools
        run: |
          sudo apt-get update
          sudo apt-get install -y cppcheck
          pip install flawfinder
      
      - name: Cppcheck
        run: cppcheck --enable=all --error-exitcode=1 src/
      
      - name: Flawfinder
        run: flawfinder --minlevel=3 src/

  asan-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build with ASan
        run: |
          gcc -fsanitize=address,undefined -g src/*.c -o programme
      
      - name: Run tests
        run: ./run_tests.sh
```

### Makefile avec options de sécurité

```makefile
CC = gcc
CFLAGS_BASE = -Wall -Wextra -Werror -std=c11

# Options de sécurité
CFLAGS_SECURE = -fstack-protector-strong \
                -D_FORTIFY_SOURCE=2 \
                -Wformat -Wformat-security \
                -fPIE

LDFLAGS_SECURE = -pie -Wl,-z,relro,-z,now

# Options de debug/analyse
CFLAGS_DEBUG = -g -O0
CFLAGS_ASAN = -fsanitize=address,undefined
CFLAGS_COVERAGE = --coverage

# Cibles
.PHONY: all release debug asan test clean analyze

all: release

release:
	$(CC) $(CFLAGS_BASE) $(CFLAGS_SECURE) -O2 src/*.c -o programme $(LDFLAGS_SECURE)

debug:
	$(CC) $(CFLAGS_BASE) $(CFLAGS_DEBUG) src/*.c -o programme_debug

asan:
	$(CC) $(CFLAGS_BASE) $(CFLAGS_DEBUG) $(CFLAGS_ASAN) src/*.c -o programme_asan

test: asan
	./programme_asan < testcases/input1.txt

analyze:
	cppcheck --enable=all src/
	flawfinder src/

clean:
	rm -f programme programme_debug programme_asan *.gcda *.gcno
```

---

## Bonnes pratiques de développement sécurisé

### Règles d'or

```c
// 1. TOUJOURS vérifier les tailles avant copie
void copier_securise(char *dest, size_t dest_size, const char *src) {
    if (strlen(src) >= dest_size) {
        // Gérer l'erreur
        return;
    }
    strcpy(dest, src);  // Maintenant sûr
}

// 2. TOUJOURS vérifier les retours de malloc
void *ptr = malloc(size);
if (ptr == NULL) {
    // Gérer l'erreur
    return ERROR;
}

// 3. TOUJOURS initialiser les pointeurs
int *p = NULL;  // Pas: int *p;

// 4. TOUJOURS mettre à NULL après free
free(p);
p = NULL;

// 5. TOUJOURS utiliser des fonctions sécurisées
snprintf(buffer, sizeof(buffer), "%s", input);  // Pas sprintf

// 6. TOUJOURS vérifier les débordements d'entiers
if (a > 0 && b > INT_MAX - a) {
    // Overflow!
}

// 7. TOUJOURS valider les entrées utilisateur
if (index < 0 || index >= array_size) {
    return ERROR;
}

// 8. JAMAIS faire confiance aux données externes
// Valider, sanitizer, vérifier les bornes

// 9. Utiliser const quand possible
void fonction(const char *lecture_seule);

// 10. Préférer les types de taille fixe pour les protocoles
#include <stdint.h>
uint32_t network_value;  // Pas: unsigned int
```

### Checklist de revue de code

```
Tous les buffers ont une taille définie
Toutes les copies vérifient la taille
Tous les malloc sont vérifiés
Tous les pointeurs sont initialisés
Tous les free sont suivis de = NULL
Pas de fonctions dangereuses (gets, strcpy, sprintf...)
Les entiers sont vérifiés avant opérations
Les entrées utilisateur sont validées
Les format strings sont constantes
Pas de race conditions évidentes
Compilé avec warnings et protections
```

### CERT C Coding Standard

Référence : [CERT C Coding Standard](https://wiki.sei.cmu.edu/confluence/display/c/)

Règles essentielles :
- **STR31-C** : Garantir que les chaînes sont terminées
- **MEM35-C** : Allouer suffisamment de mémoire
- **INT32-C** : Vérifier les débordements
- **FIO30-C** : Exclure les entrées utilisateur des format strings
- **ERR33-C** : Détecter et gérer les erreurs

---

## Exercices du chapitre

### Exercice 10.1 : Identifier les vulnérabilités (Débutant)

Trouvez toutes les vulnérabilités dans ce code :

```c
#include <stdio.h>
#include <string.h>

void traiter(char *input) {
    char buffer[64];
    char *ptr;
    
    strcpy(buffer, input);
    printf(buffer);
    
    ptr = malloc(strlen(input));
    strcpy(ptr, input);
    
    free(ptr);
    printf("Longueur: %d\n", strlen(ptr));
}

int main(int argc, char *argv[]) {
    if (argc > 1) {
        traiter(argv[1]);
    }
    return 0;
}
```

### Exercice 10.2 : Corriger le code (Intermédiaire)

Corrigez le code de l'exercice 10.1 en appliquant les bonnes pratiques.

### Exercice 10.3 : Overflow d'entier (Intermédiaire)

Écrivez une fonction `safe_multiply` qui multiplie deux `size_t` et retourne -1 en cas d'overflow :

```c
int safe_multiply(size_t a, size_t b, size_t *result);
```

### Exercice 10.4 : Allocation sécurisée (Intermédiaire)

Implémentez :
- `void *secure_malloc(size_t size)` : malloc avec vérification
- `void *secure_calloc(size_t nmemb, size_t size)` : avec vérification overflow
- `void secure_free(void **ptr)` : free + NULL

### Exercice 10.5 : Analyse de binaire (Avancé)

Utilisez les outils pour analyser un binaire :
1. `checksec` pour vérifier les protections
2. `objdump` pour désassembler
3. Identifiez les fonctions dangereuses avec `nm` et `strings`

### Exercice 10.6 : Pipeline CI/CD (Avancé)

Créez un pipeline CI/CD complet pour un projet C incluant :
- Compilation avec warnings
- Analyse statique (cppcheck, flawfinder)
- Tests avec ASan
- Rapport de couverture

---

## Résumé du chapitre

**Vulnérabilités majeures** : Buffer overflow, format string, integer overflow, use-after-free

**Fonctions dangereuses** : gets, strcpy, sprintf → Utiliser les alternatives sécurisées

**Protections système** : ASLR, DEP/NX, Stack Canaries, RELRO

**Analyse statique** : Cppcheck, Clang Static Analyzer, Flawfinder, Coverity

**Analyse dynamique** : Valgrind, AddressSanitizer, Fuzzing (AFL, libFuzzer)

**DevSecOps** : Intégrer la sécurité dans CI/CD, automatiser les analyses

**Bonnes pratiques** : Valider, vérifier, initialiser, libérer proprement

---

## Ressources supplémentaires

- [OWASP Secure Coding Practices](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/)
- [CERT C Coding Standard](https://wiki.sei.cmu.edu/confluence/display/c/)
- [CWE Top 25](https://cwe.mitre.org/top25/)
- [Smashing the Stack for Fun and Profit](http://phrack.org/issues/49/14.html) (article historique)
- [Modern Binary Exploitation](https://github.com/RPISEC/MBE) (cours)

---

## Conclusion du cours

Félicitations ! Vous avez parcouru les fondamentaux du langage C, de la syntaxe de base jusqu'aux considérations de sécurité avancées.

### Récapitulatif des 10 chapitres

1. **Introduction** : Histoire, environnement, compilation
2. **Variables et types** : Typage statique, constantes, entrées/sorties
3. **Opérateurs** : Arithmétiques, logiques, bit-à-bit
4. **Structures de contrôle** : Conditions, boucles, sauts
5. **Fonctions** : Modularité, récursivité, pointeurs de fonctions
6. **Tableaux et chaînes** : Manipulation, sécurité des chaînes
7. **Pointeurs** : Concept fondamental, arithmétique, erreurs courantes
8. **Structures et unions** : Types composés, enum, champs de bits
9. **Mémoire et fichiers** : Allocation dynamique, I/O fichiers
10. **Cybersécurité** : Vulnérabilités, protections, DevSecOps

### Prochaines étapes

- Pratiquez avec des projets réels
- Contribuez à des projets open-source en C
- Explorez les domaines spécifiques : embarqué, systèmes, sécurité
- Apprenez des langages complémentaires : Rust (sécurité), C++ (OOP)

Bonne programmation en C !
