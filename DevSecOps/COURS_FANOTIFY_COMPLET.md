# 📚 Cours Complet : FANOTIFY en C

## Guide de Référence pour le Projet Sentinel HIDS

> **Environnement cible** : Kali Linux, Kernel 6.16.8+  
> **Niveau** : Débutant → Intermédiaire  
> **Projet** : HIDS (Host-based Intrusion Detection System)

---

# Table des Matières

1. [PARTIE 1 : Rappels des Bases du C](#partie-1--rappels-des-bases-du-c)
2. [PARTIE 2 : Introduction à Fanotify](#partie-2--introduction-à-fanotify)
3. [PARTIE 3 : L'API Fanotify en Détail](#partie-3--lapi-fanotify-en-détail)
4. [PARTIE 4 : Exemples Pratiques Progressifs](#partie-4--exemples-pratiques-progressifs)
5. [PARTIE 5 : Application au Projet HIDS](#partie-5--application-au-projet-hids)
6. [PARTIE 6 : Référence Rapide](#partie-6--référence-rapide)

---

# PARTIE 1 : Rappels des Bases du C

## 1.1 Types de données fondamentaux

### Les types de base

```c
/*
 * En C, chaque variable a un TYPE qui détermine :
 * - Sa taille en mémoire (en octets)
 * - Les valeurs qu'elle peut contenir
 * - Les opérations possibles dessus
 */

#include <stdio.h>      /* Pour printf */
#include <stdint.h>     /* Pour les types à taille fixe */
#include <stdbool.h>    /* Pour bool, true, false (C99+) */

int main(void)
{
    /* ═══ ENTIERS SIGNÉS (peuvent être négatifs) ═══ */
    
    char c = 'A';           /* 1 octet, -128 à 127 */
    short s = 1000;         /* 2 octets minimum */
    int i = 42;             /* 4 octets généralement */
    long l = 100000L;       /* 4 ou 8 octets selon système */
    long long ll = 9000000000000000000LL;  /* 8 octets */

    /* ═══ ENTIERS NON SIGNÉS (positifs uniquement) ═══ */
    
    unsigned char uc = 255;           /* 0 à 255 */
    unsigned int ui = 4294967295U;    /* 0 à ~4 milliards */
    
    /* ═══ TYPES À TAILLE FIXE (recommandés pour la portabilité) ═══ */
    
    int8_t   i8  = -100;      /* Exactement 8 bits signés */
    uint8_t  u8  = 200;       /* Exactement 8 bits non signés */
    int32_t  i32 = -50000;    /* Exactement 32 bits signés */
    uint32_t u32 = 50000;     /* Exactement 32 bits non signés */
    int64_t  i64 = -1000000;  /* Exactement 64 bits signés */
    uint64_t u64 = 1000000;   /* Exactement 64 bits non signés */
    
    /* ═══ TYPES SPÉCIAUX SYSTÈME ═══ */
    
    size_t taille = 1024;     /* Taille/indice, toujours positif */
    ssize_t resultat = -1;    /* Taille signée (retours de read/write) */
    pid_t pid = 1234;         /* ID de processus */
    uid_t uid = 1000;         /* ID utilisateur */
    
    /* ═══ BOOLÉENS (C99+) ═══ */
    
    bool actif = true;        /* Soit true (1) soit false (0) */
    bool desactive = false;
    
    return 0;
}
```

### Pourquoi utiliser les types à taille fixe ?

```c
/*
 * PROBLÈME : La taille de 'int' varie selon les systèmes
 * SOLUTION : Utiliser <stdint.h> pour des types à taille garantie
 * 
 * Pour fanotify, c'est CRUCIAL car les structures du kernel
 * ont des tailles fixes définies.
 */

#include <stdint.h>

/* Exemple : Structure avec tailles garanties */
struct mon_evenement {
    uint32_t type;      /* Toujours 4 octets */
    uint64_t timestamp; /* Toujours 8 octets */
    int32_t  pid;       /* Toujours 4 octets */
};
```

---

## 1.2 Les pointeurs

### Concept fondamental

```c
/*
 * Un POINTEUR est une variable qui stocke une ADRESSE MÉMOIRE.
 * 
 * Analogie : 
 * - Une variable normale est comme une maison (elle contient quelque chose)
 * - Un pointeur est comme un GPS qui stocke l'adresse de la maison
 */

#include <stdio.h>

int main(void)
{
    /* ═══ DÉCLARATION ET INITIALISATION ═══ */
    
    int valeur = 42;        /* Une variable normale contenant 42 */
    int *pointeur;          /* Déclaration d'un pointeur vers un int */
                            /* L'étoile (*) signifie "pointeur vers" */
    
    pointeur = &valeur;     /* L'esperluette (&) donne l'ADRESSE de valeur */
                            /* Maintenant pointeur "pointe vers" valeur */
    
    /* ═══ LECTURE VIA POINTEUR ═══ */
    
    printf("Valeur directe    : %d\n", valeur);      /* 42 */
    printf("Adresse de valeur : %p\n", (void*)&valeur);
    printf("Contenu du pointeur : %p\n", (void*)pointeur);
    printf("Valeur pointée (*pointeur) : %d\n", *pointeur);  /* 42 */
    
    /* ═══ MODIFICATION VIA POINTEUR ═══ */
    
    *pointeur = 100;        /* Modifie la valeur à l'adresse pointée */
    printf("Après modification : valeur = %d\n", valeur);  /* 100 ! */
    
    /* ═══ POINTEUR NULL ═══ */
    
    int *ptr_null = NULL;   /* Pointeur qui ne pointe vers rien */
                            /* TOUJOURS initialiser à NULL si pas d'adresse */
    
    /* Vérification avant utilisation (OBLIGATOIRE) */
    if (ptr_null != NULL) {
        printf("Valeur : %d\n", *ptr_null);  /* Ne sera pas exécuté */
    }
    
    return 0;
}
```

### Pointeurs et fonctions

```c
/*
 * En C, les arguments sont passés PAR VALEUR par défaut.
 * Pour modifier une variable depuis une fonction, on passe son ADRESSE.
 */

#include <stdio.h>

/* ═══ PASSAGE PAR VALEUR (copie) ═══ */
void doubler_valeur(int x)
{
    x = x * 2;  /* Modifie la COPIE locale, pas l'original */
}

/* ═══ PASSAGE PAR POINTEUR (référence) ═══ */
void doubler_pointeur(int *x)
{
    *x = *x * 2;  /* Modifie la valeur à L'ADRESSE pointée */
}

int main(void)
{
    int n = 5;
    
    doubler_valeur(n);
    printf("Après doubler_valeur : n = %d\n", n);  /* Toujours 5 ! */
    
    doubler_pointeur(&n);  /* On passe L'ADRESSE de n */
    printf("Après doubler_pointeur : n = %d\n", n);  /* 10 ! */
    
    return 0;
}
```

---

## 1.3 Les structures

### Définition et utilisation

```c
/*
 * Une STRUCTURE permet de regrouper plusieurs variables de types différents
 * sous un même nom. C'est essentiel pour organiser des données complexes.
 * 
 * fanotify utilise BEAUCOUP de structures pour communiquer avec le kernel.
 */

#include <stdio.h>
#include <string.h>

/* ═══ DÉFINITION D'UNE STRUCTURE ═══ */

/* Méthode 1 : struct + nom */
struct personne {
    char nom[50];
    int age;
    float taille;
};

/* Méthode 2 : typedef (crée un alias de type) - RECOMMANDÉ */
typedef struct {
    char nom[50];
    int age;
    float taille;
} Personne;  /* Maintenant "Personne" est un type comme "int" */

/* ═══ UTILISATION ═══ */

int main(void)
{
    /* Déclaration et initialisation */
    Personne alice = {"Alice Dupont", 30, 1.65f};
    
    /* Accès aux membres avec le point (.) */
    printf("Nom    : %s\n", alice.nom);
    printf("Age    : %d ans\n", alice.age);
    printf("Taille : %.2f m\n", alice.taille);
    
    /* Modification */
    alice.age = 31;
    strcpy(alice.nom, "Alice Martin");
    
    /* ═══ AVEC UN POINTEUR ═══ */
    
    Personne *ptr = &alice;
    
    /* Deux façons d'accéder aux membres via pointeur */
    printf("Via (*ptr).age : %d\n", (*ptr).age);
    printf("Via ptr->age   : %d\n", ptr->age);  /* NOTATION FLÈCHE - Préférée */
    
    /* La flèche (->) combine le déréférencement et l'accès au membre */
    
    return 0;
}
```

---

## 1.4 Allocation mémoire dynamique

### malloc, free et les bonnes pratiques

```c
/*
 * L'allocation DYNAMIQUE permet de réserver de la mémoire pendant
 * l'exécution du programme, quand on ne connaît pas la taille à l'avance.
 */

#include <stdio.h>
#include <stdlib.h>  /* Pour malloc, free, etc. */
#include <string.h>

int main(void)
{
    /* ═══ ALLOCATION D'UN ENTIER ═══ */
    
    int *ptr_int = malloc(sizeof(int));
    
    /* TOUJOURS vérifier si l'allocation a réussi */
    if (ptr_int == NULL) {
        fprintf(stderr, "Erreur: allocation mémoire échouée\n");
        return 1;
    }
    
    *ptr_int = 42;
    printf("Valeur allouée : %d\n", *ptr_int);
    
    free(ptr_int);      /* LIBÉRER quand on n'en a plus besoin */
    ptr_int = NULL;     /* Bonne pratique : mettre à NULL après free */
    
    /* ═══ ALLOCATION D'UN TABLEAU ═══ */
    
    int nb_elements = 10;
    int *tableau = malloc(nb_elements * sizeof(int));
    
    if (tableau == NULL) {
        fprintf(stderr, "Erreur: allocation tableau échouée\n");
        return 1;
    }
    
    /* Remplissage */
    for (int i = 0; i < nb_elements; i++) {
        tableau[i] = i * 10;
    }
    
    printf("tableau[5] = %d\n", tableau[5]);  /* 50 */
    
    free(tableau);
    tableau = NULL;
    
    /* ═══ CALLOC : ALLOCATION + INITIALISATION À ZÉRO ═══ */
    
    int *zeros = calloc(10, sizeof(int));  /* 10 entiers initialisés à 0 */
    
    if (zeros == NULL) {
        return 1;
    }
    
    printf("zeros[5] = %d\n", zeros[5]);  /* 0 (garanti) */
    
    free(zeros);
    
    return 0;
}
```

### Erreurs courantes et comment les éviter

```c
/*
 * ERREURS FRÉQUENTES AVEC LA MÉMOIRE DYNAMIQUE
 */

/* ERREUR 1 : Oublier de vérifier le retour de malloc */
// MAUVAIS : int *p = malloc(1000000000000); *p = 42; /* CRASH ! */

/* ERREUR 2 : Utiliser après free (Use After Free) */
// MAUVAIS : free(p); printf("%d\n", *p); /* Comportement indéfini */

/* ERREUR 3 : Double free */
// MAUVAIS : free(p); free(p); /* CRASH ! */

/* ERREUR 4 : Fuite mémoire (Memory Leak) */
// MAUVAIS : void f() { int *p = malloc(1000); } /* Jamais libéré ! */

/* SOLUTION : Toujours mettre à NULL après free */
free(p);
p = NULL;
free(p);  /* OK, free(NULL) ne fait rien */
```

---

## 1.5 Les macros préprocesseur

```c
/*
 * Le PRÉPROCESSEUR traite le code AVANT la compilation.
 * Les macros sont utilisées massivement dans les headers système.
 */

#include <stdio.h>

/* ═══ CONSTANTES ═══ */

#define MAX_PATH 4096
#define VERSION "1.0.0"
#define PI 3.14159

/* ═══ MACROS FONCTIONNELLES ═══ */

#define CARRE(x) ((x) * (x))  /* Parenthèses importantes ! */

/* Macro avec plusieurs lignes */
#define LOG_ERROR(msg) do { \
    fprintf(stderr, "[ERROR] %s:%d: %s\n", __FILE__, __LINE__, msg); \
} while(0)

/* ═══ MACROS CONDITIONNELLES ═══ */

#define DEBUG 1

#if DEBUG
    #define DEBUG_PRINT(fmt, ...) printf("[DEBUG] " fmt "\n", ##__VA_ARGS__)
#else
    #define DEBUG_PRINT(fmt, ...)  /* Ne fait rien */
#endif

/* ═══ PROTECTION DES HEADERS (Include Guards) ═══ */

/*
 * Dans un fichier .h :
 * 
 * #ifndef MON_HEADER_H
 * #define MON_HEADER_H
 * // Contenu du header...
 * #endif
 */

/* ═══ MACROS PRÉDÉFINIES UTILES ═══ */

int main(void)
{
    printf("Fichier : %s\n", __FILE__);
    printf("Ligne   : %d\n", __LINE__);
    printf("Date    : %s\n", __DATE__);
    printf("Fonction: %s\n", __func__);
    
    LOG_ERROR("Ceci est une erreur de test");
    DEBUG_PRINT("Variable x = %d", 42);
    
    return 0;
}
```

---

## 1.6 Gestion des erreurs avec errno

```c
/*
 * En C et POSIX, les fonctions système signalent les erreurs via :
 * 1. Une valeur de retour spéciale (souvent -1 ou NULL)
 * 2. La variable globale 'errno' qui contient le CODE d'erreur
 */

#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <fcntl.h>
#include <unistd.h>

int main(void)
{
    /* Tenter d'ouvrir un fichier qui n'existe pas */
    int fd = open("/fichier/inexistant/test.txt", O_RDONLY);
    
    if (fd == -1) {  /* -1 indique une erreur */
        /* errno contient le code d'erreur */
        printf("Code errno : %d\n", errno);
        
        /* strerror() convertit le code en message lisible */
        printf("Message    : %s\n", strerror(errno));
        
        /* perror() affiche directement le message avec un préfixe */
        perror("Erreur open");
        
        /* Codes d'erreur courants */
        switch (errno) {
            case ENOENT:
                printf("Fichier inexistant (ENOENT)\n");
                break;
            case EACCES:
                printf("Permission refusée (EACCES)\n");
                break;
            case ENOMEM:
                printf("Mémoire insuffisante (ENOMEM)\n");
                break;
            case EINVAL:
                printf("Argument invalide (EINVAL)\n");
                break;
        }
    } else {
        close(fd);
    }
    
    return 0;
}
```

---

# PARTIE 2 : Introduction à Fanotify

## 2.1 Qu'est-ce que fanotify ?

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           QU'EST-CE QUE FANOTIFY ?                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  fanotify = "File Access NOTIFication"                                      │
│                                                                             │
│  C'est une API du noyau Linux (depuis 2.6.37) qui permet à un programme     │
│  en espace utilisateur de :                                                 │
│                                                                             │
│  1. SURVEILLER les accès aux fichiers (lecture, écriture, ouverture...)     │
│  2. BLOQUER ou AUTORISER ces accès (antivirus, contrôle d'accès)            │
│                                                                             │
│  ┌────────────┐    événement     ┌────────────┐                             │
│  │ Processus  │ ---------------> |  Kernel    │                             │
│  │ (ex: vim)  │    open()        │  Linux     │                             │
│  └────────────┘                  └─────┬──────┘                             │
│                                        │                                    │
│                                        │ notification                       │
│                                        ▼                                    │
│                                  ┌────────────┐                             │
│                                  │ Votre HIDS │                             │
│                                  │ (fanotify) │                             │
│                                  └────────────┘                             │
│                                                                             │
│  Cas d'usage typiques :                                                     │
│  • Antivirus (scanner les fichiers à l'ouverture)                           │
│  • HIDS (détecter les modifications suspectes)                              │
│  • Audit (journaliser les accès)                                            │
│  • DLP (Data Loss Prevention)                                               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2.2 Deux modes de fonctionnement

```
┌──────────────────────────────────────────────────────────────────────┐
│                    DEUX MODES DE FONCTIONNEMENT                      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────────┐  ┌──────────────────────────────┐  │
│  │     MODE NOTIFICATION        │  │     MODE PERMISSION          │  │
│  ├──────────────────────────────┤  ├──────────────────────────────┤  │
│  │                              │  │                              │  │
│  │  • Reçoit les événements     │  │  • Reçoit les événements     │  │
│  │    APRÈS leur exécution      │  │    AVANT leur exécution      │  │
│  │                              │  │                              │  │
│  │  • Ne peut PAS bloquer       │  │  • Peut AUTORISER ou BLOQUER │  │
│  │                              │  │                              │  │
│  │  • Plus simple à utiliser    │  │  • Plus complexe             │  │
│  │                              │  │                              │  │
│  │  • Idéal pour : audit,       │  │  • Idéal pour : antivirus,   │  │
│  │    journalisation, backup    │  │    contrôle d'accès          │  │
│  │                              │  │                              │  │
│  │  Flags : FAN_ACCESS,         │  │  Flags : FAN_OPEN_PERM,      │  │
│  │          FAN_MODIFY,         │  │          FAN_ACCESS_PERM,    │  │
│  │          FAN_CLOSE_WRITE     │  │          FAN_OPEN_EXEC_PERM  │  │
│  │                              │  │                              │  │
│  └──────────────────────────────┘  └──────────────────────────────┘  │
│                                                                      │
│  POUR SENTINEL HIDS :                                                │
│  → On utilisera les DEUX modes                                       │
│  → Notification pour les logs/audit                                  │
│  → Permission pour bloquer les accès suspects                        │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

## 2.3 Fanotify vs Inotify

```
┌───────────────────────────────────────────────────────┐
│                  FANOTIFY vs INOTIFY                  │
├───────────────────────────────────────────────────────┤
│                                                       │
│  ┌────────────────────────┬────────────────────────┐  │
│  │       INOTIFY          │       FANOTIFY         │  │
│  ├────────────────────────┼────────────────────────┤  │
│  │ Depuis kernel 2.6.13   │ Depuis kernel 2.6.37   │  │
│  ├────────────────────────┼────────────────────────┤  │
│  │ Surveillance par       │ Surveillance par       │  │
│  │ FICHIER/RÉPERTOIRE     │ MOUNT POINT ou         │  │
│  │ (un watch par fichier) │ FILESYSTEM entier      │  │
│  ├────────────────────────┼────────────────────────┤  │
│  │ Pas d'info sur le      │ FOURNIT le PID du      │  │
│  │ processus responsable  │ processus responsable  │  │
│  ├────────────────────────┼────────────────────────┤  │
│  │ Mode notification      │ Mode notification ET   │  │
│  │ UNIQUEMENT             │ mode PERMISSION        │  │
│  ├────────────────────────┼────────────────────────┤  │
│  │ Pas besoin de          │ Nécessite CAP_SYS_ADMIN│  │
│  │ privilèges spéciaux    │ ou root                │  │
│  ├────────────────────────┼────────────────────────┤  │
│  │ Bon pour : petit       │ Bon pour : surveillance│  │
│  │ nombre de fichiers     │ système, HIDS,         │  │
│  │                        │ antivirus              │  │
│  └────────────────────────┴────────────────────────┘  │
│                                                       │
│  POUR UN HIDS : fanotify est le SEUL CHOIX VIABLE     │
│  - On veut savoir QUI accède aux fichiers (PID)       │
│  - On veut potentiellement BLOQUER les accès          │
│  - On surveille tout le système                       │
│                                                       │
└───────────────────────────────────────────────────────┘
```

## 2.4 Permissions et capabilities

```c
/*
 * FANOTIFY NÉCESSITE DES PRIVILÈGES ÉLEVÉS
 * 
 * Deux options :
 * 1. Exécuter en tant que root (simple mais moins sécurisé)
 * 2. Utiliser les capabilities Linux (recommandé)
 * 
 * Capabilities nécessaires :
 * - CAP_SYS_ADMIN : Requis pour fanotify_init() et fanotify_mark()
 */

/* Vérifier si on a les privilèges nécessaires */

#include <stdio.h>
#include <unistd.h>

int verifier_privileges(void)
{
    if (geteuid() == 0) {
        printf("Exécution en tant que root ✓\n");
        return 1;
    }
    
    printf("Privilèges insuffisants ✗\n");
    printf("Lancez avec : sudo ./programme\n");
    return 0;
}
```

```bash
# Méthode 1 : Capability sur l'exécutable (recommandé)
sudo setcap cap_sys_admin+ep ./sentinel

# Méthode 2 : Exécuter avec sudo
sudo ./sentinel
```

---

# PARTIE 3 : L'API Fanotify en Détail

## 3.1 fanotify_init()

```c
/*
 * fanotify_init() - Crée un nouveau groupe de notification fanotify
 * 
 * int fanotify_init(unsigned int flags, unsigned int event_f_flags);
 * 
 * RETOUR :
 *     Succès : descripteur de fichier (fd) >= 0
 *     Échec  : -1, errno contient le code d'erreur
 */

#include <stdio.h>
#include <fcntl.h>
#include <sys/fanotify.h>
#include <errno.h>
#include <unistd.h>

int main(void)
{
    int fd;
    
    /*
     * ═══ FLAGS DE CLASSE (obligatoire, en choisir UN) ═══
     * 
     * FAN_CLASS_NOTIF (0x00)
     * - Mode notification uniquement
     * - Reçoit les événements APRÈS leur exécution
     * 
     * FAN_CLASS_CONTENT (0x04)
     * - Mode permission pour le contenu
     * - Peut recevoir FAN_OPEN_PERM, FAN_ACCESS_PERM
     * 
     * FAN_CLASS_PRE_CONTENT (0x08)
     * - Mode permission pré-contenu
     */
    
    /*
     * ═══ FLAGS DE COMPORTEMENT (optionnels) ═══
     * 
     * FAN_CLOEXEC     - Ferme automatiquement le fd lors d'un exec()
     * FAN_NONBLOCK    - read() ne bloque pas si pas d'événement
     * FAN_UNLIMITED_QUEUE - Pas de limite sur la queue d'événements
     * FAN_UNLIMITED_MARKS - Pas de limite sur le nombre de marks
     * FAN_REPORT_FID  - Rapporte le File ID au lieu du fd
     */
    
    /* EXEMPLE 1 : Surveillance simple (notification uniquement) */
    fd = fanotify_init(
        FAN_CLASS_NOTIF | FAN_CLOEXEC,
        O_RDONLY | O_CLOEXEC
    );
    
    if (fd == -1) {
        perror("fanotify_init");
        /* EPERM  : Pas les privilèges */
        /* ENOMEM : Plus de mémoire */
        /* ENOSYS : fanotify non supporté */
        return 1;
    }
    printf("Mode notification : fd = %d\n", fd);
    close(fd);
    
    /* EXEMPLE 2 : Contrôle d'accès (mode permission) */
    fd = fanotify_init(
        FAN_CLASS_CONTENT | FAN_CLOEXEC | FAN_NONBLOCK,
        O_RDONLY | O_LARGEFILE | O_CLOEXEC
    );
    
    if (fd == -1) {
        perror("fanotify_init");
        return 1;
    }
    printf("Mode permission : fd = %d\n", fd);
    close(fd);
    
    return 0;
}
```

---

## 3.2 fanotify_mark()

```c
/*
 * fanotify_mark() - Ajoute, modifie ou supprime un mark fanotify
 * 
 * int fanotify_mark(int fanotify_fd, unsigned int flags,
 *                   uint64_t mask, int dirfd, const char *pathname);
 * 
 * Un "mark" définit :
 *     - QUOI surveiller (fichier, répertoire, mount point)
 *     - QUELS événements capturer
 */

#include <stdio.h>
#include <fcntl.h>
#include <sys/fanotify.h>
#include <unistd.h>

int main(void)
{
    int fd = fanotify_init(FAN_CLASS_NOTIF | FAN_CLOEXEC, O_RDONLY);
    if (fd == -1) return 1;
    
    int ret;
    
    /*
     * ═══ FLAGS D'ACTION ═══
     * FAN_MARK_ADD    - Ajoute les événements
     * FAN_MARK_REMOVE - Retire les événements
     * FAN_MARK_FLUSH  - Supprime TOUS les marks
     */
    
    /*
     * ═══ FLAGS DE TYPE ═══
     * (aucun)             - Mark sur un INODE
     * FAN_MARK_MOUNT      - Mark sur un POINT DE MONTAGE
     * FAN_MARK_FILESYSTEM - Mark sur un FILESYSTEM entier
     */
    
    /*
     * ═══ ÉVÉNEMENTS DE NOTIFICATION ═══
     * FAN_ACCESS       - Fichier lu
     * FAN_MODIFY       - Fichier modifié
     * FAN_CLOSE_WRITE  - Fermé après écriture
     * FAN_CLOSE_NOWRITE- Fermé sans écriture
     * FAN_OPEN         - Fichier ouvert
     * FAN_OPEN_EXEC    - Ouvert pour exécution
     * FAN_ATTRIB       - Métadonnées modifiées
     * FAN_CREATE       - Fichier créé
     * FAN_DELETE       - Fichier supprimé
     * FAN_MOVE         - Fichier déplacé
     */
    
    /*
     * ═══ ÉVÉNEMENTS DE PERMISSION ═══
     * FAN_OPEN_PERM      - Permission avant ouverture
     * FAN_ACCESS_PERM    - Permission avant lecture
     * FAN_OPEN_EXEC_PERM - Permission avant exécution
     */
    
    /* EXEMPLE 1 : Surveiller un fichier spécifique */
    ret = fanotify_mark(
        fd,
        FAN_MARK_ADD,
        FAN_OPEN | FAN_CLOSE_WRITE,
        AT_FDCWD,
        "/etc/passwd"
    );
    if (ret == 0) printf("Mark sur /etc/passwd ✓\n");
    
    /* EXEMPLE 2 : Surveiller tout un point de montage */
    ret = fanotify_mark(
        fd,
        FAN_MARK_ADD | FAN_MARK_MOUNT,
        FAN_MODIFY | FAN_CLOSE_WRITE,
        AT_FDCWD,
        "/var/log"
    );
    if (ret == 0) printf("Mark sur /var/log (mount) ✓\n");
    
    /* EXEMPLE 3 : Surveiller tout le système */
    ret = fanotify_mark(
        fd,
        FAN_MARK_ADD | FAN_MARK_MOUNT,
        FAN_OPEN | FAN_CLOSE_WRITE | FAN_MODIFY,
        AT_FDCWD,
        "/"
    );
    if (ret == 0) printf("Mark sur / ✓\n");
    
    /* EXEMPLE 4 : Ignorer un fichier (whitelist) */
    ret = fanotify_mark(
        fd,
        FAN_MARK_ADD | FAN_MARK_IGNORED_MASK,
        FAN_OPEN | FAN_CLOSE_WRITE | FAN_MODIFY,
        AT_FDCWD,
        "/var/log/syslog"
    );
    if (ret == 0) printf("Ignore /var/log/syslog ✓\n");
    
    close(fd);
    return 0;
}
```

---

## 3.3 Lecture des événements

```c
/*
 * Structure fanotify_event_metadata
 * 
 * struct fanotify_event_metadata {
 *     __u32 event_len;    // Taille totale de l'événement
 *     __u8 vers;          // Version (FANOTIFY_METADATA_VERSION)
 *     __u8 reserved;      // Réservé
 *     __u16 metadata_len; // Taille de cette structure
 *     __aligned_u64 mask; // Événements (FAN_OPEN, etc.)
 *     __s32 fd;           // fd vers le fichier
 *     __s32 pid;          // PID du processus
 * };
 */

#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/fanotify.h>
#include <errno.h>
#include <signal.h>
#include <limits.h>

static volatile int continuer = 1;

void handler_sigint(int sig) {
    (void)sig;
    continuer = 0;
}

/* Obtenir le chemin du fichier à partir d'un fd */
char *obtenir_chemin(int fd, char *buffer, size_t taille)
{
    char fd_path[64];
    snprintf(fd_path, sizeof(fd_path), "/proc/self/fd/%d", fd);
    ssize_t len = readlink(fd_path, buffer, taille - 1);
    if (len == -1) return NULL;
    buffer[len] = '\0';
    return buffer;
}

int main(void)
{
    signal(SIGINT, handler_sigint);
    
    /* Initialiser fanotify */
    int fanotify_fd = fanotify_init(
        FAN_CLASS_NOTIF | FAN_CLOEXEC,
        O_RDONLY | O_LARGEFILE | O_CLOEXEC
    );
    
    if (fanotify_fd == -1) {
        perror("fanotify_init");
        return 1;
    }
    
    /* Marquer /tmp */
    if (fanotify_mark(fanotify_fd, FAN_MARK_ADD | FAN_MARK_MOUNT,
                      FAN_OPEN | FAN_CLOSE_WRITE | FAN_MODIFY,
                      AT_FDCWD, "/tmp") == -1) {
        perror("fanotify_mark");
        close(fanotify_fd);
        return 1;
    }
    
    printf("Surveillance de /tmp (Ctrl+C pour arrêter)\n\n");
    
    char buffer[4096];
    
    while (continuer) {
        ssize_t len = read(fanotify_fd, buffer, sizeof(buffer));
        
        if (len == -1) {
            if (errno == EINTR) continue;
            perror("read");
            break;
        }
        
        /* Parcourir les événements */
        struct fanotify_event_metadata *metadata;
        metadata = (struct fanotify_event_metadata *)buffer;
        
        while (FAN_EVENT_OK(metadata, len)) {
            if (metadata->vers != FANOTIFY_METADATA_VERSION) {
                fprintf(stderr, "Version non supportée\n");
                break;
            }
            
            /* Obtenir le chemin */
            char chemin[PATH_MAX];
            if (metadata->fd >= 0) {
                if (!obtenir_chemin(metadata->fd, chemin, sizeof(chemin))) {
                    snprintf(chemin, sizeof(chemin), "[fd=%d]", metadata->fd);
                }
            } else {
                snprintf(chemin, sizeof(chemin), "[no fd]");
            }
            
            /* Afficher l'événement */
            printf("[PID %5d] ", metadata->pid);
            if (metadata->mask & FAN_OPEN) printf("OPEN ");
            if (metadata->mask & FAN_MODIFY) printf("MODIFY ");
            if (metadata->mask & FAN_CLOSE_WRITE) printf("CLOSE_W ");
            printf("%s\n", chemin);
            
            /* IMPORTANT : Fermer le fd */
            if (metadata->fd >= 0) close(metadata->fd);
            
            metadata = FAN_EVENT_NEXT(metadata, len);
        }
    }
    
    close(fanotify_fd);
    printf("\nArrêt.\n");
    return 0;
}
```

---

## 3.4 Répondre aux événements de permission

```c
/*
 * Pour les événements FAN_*_PERM, on DOIT répondre au kernel.
 * Si on ne répond pas, le processus reste BLOQUÉ !
 * 
 * struct fanotify_response {
 *     __s32 fd;       // Le fd de l'événement
 *     __u32 response; // FAN_ALLOW ou FAN_DENY
 * };
 */

#define _GNU_SOURCE
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/fanotify.h>
#include <signal.h>
#include <limits.h>

static volatile int continuer = 1;

void handler_sigint(int sig) { (void)sig; continuer = 0; }

char *obtenir_chemin(int fd, char *buf, size_t sz) {
    char p[64];
    snprintf(p, sizeof(p), "/proc/self/fd/%d", fd);
    ssize_t l = readlink(p, buf, sz - 1);
    if (l == -1) return NULL;
    buf[l] = '\0';
    return buf;
}

/* Blacklist simple */
int est_bloque(const char *chemin) {
    if (strcmp(chemin, "/tmp/interdit.txt") == 0) return 1;
    if (strcmp(chemin, "/tmp/secret.dat") == 0) return 1;
    return 0;
}

int main(void)
{
    signal(SIGINT, handler_sigint);
    
    /* Mode PERMISSION */
    int fd = fanotify_init(FAN_CLASS_CONTENT | FAN_CLOEXEC,
                           O_RDONLY | O_LARGEFILE | O_CLOEXEC);
    if (fd == -1) { perror("fanotify_init"); return 1; }
    
    /* Marquer avec permission */
    if (fanotify_mark(fd, FAN_MARK_ADD | FAN_MARK_MOUNT,
                      FAN_OPEN_PERM, AT_FDCWD, "/tmp") == -1) {
        perror("fanotify_mark");
        close(fd);
        return 1;
    }
    
    printf("Contrôle d'accès sur /tmp\n");
    printf("Fichiers bloqués: /tmp/interdit.txt, /tmp/secret.dat\n\n");
    
    char buffer[4096];
    
    while (continuer) {
        ssize_t len = read(fd, buffer, sizeof(buffer));
        if (len == -1) { if (errno == EINTR) continue; break; }
        
        struct fanotify_event_metadata *meta =
            (struct fanotify_event_metadata *)buffer;
        
        while (FAN_EVENT_OK(meta, len)) {
            if (meta->vers != FANOTIFY_METADATA_VERSION) break;
            
            if (meta->mask & FAN_OPEN_PERM) {
                char chemin[PATH_MAX] = "[?]";
                if (meta->fd >= 0)
                    obtenir_chemin(meta->fd, chemin, sizeof(chemin));
                
                /* Décision */
                struct fanotify_response response;
                response.fd = meta->fd;
                
                if (est_bloque(chemin)) {
                    response.response = FAN_DENY;
                    printf("\033[31m[BLOQUÉ]\033[0m  PID %d: %s\n",
                           meta->pid, chemin);
                } else {
                    response.response = FAN_ALLOW;
                    printf("\033[32m[AUTORISÉ]\033[0m PID %d: %s\n",
                           meta->pid, chemin);
                }
                
                /* ENVOYER LA RÉPONSE */
                write(fd, &response, sizeof(response));
            }
            
            if (meta->fd >= 0) close(meta->fd);
            meta = FAN_EVENT_NEXT(meta, len);
        }
    }
    
    close(fd);
    return 0;
}
```

---

# PARTIE 4 : Exemples Pratiques

## Programme complet de surveillance

```c
/*
 * Moniteur fanotify complet avec filtrage
 */

#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/fanotify.h>
#include <errno.h>
#include <signal.h>
#include <limits.h>
#include <time.h>

#define BUFFER_SIZE 65536

static volatile sig_atomic_t g_running = 1;

/* Fichiers critiques */
static const char *CRITICAL_FILES[] = {
    "/etc/passwd", "/etc/shadow", "/etc/sudoers",
    "/etc/ssh/sshd_config", "/var/log/auth.log", NULL
};

/* Répertoires à ignorer */
static const char *IGNORED_DIRS[] = {
    "/proc", "/sys", "/dev", "/run", NULL
};

static void signal_handler(int sig) { (void)sig; g_running = 0; }

static int get_file_path(int fd, char *path, size_t sz) {
    char p[32];
    snprintf(p, sizeof(p), "/proc/self/fd/%d", fd);
    ssize_t l = readlink(p, path, sz - 1);
    if (l == -1) return -1;
    path[l] = '\0';
    return 0;
}

static int should_ignore(const char *path) {
    for (int i = 0; IGNORED_DIRS[i]; i++) {
        size_t len = strlen(IGNORED_DIRS[i]);
        if (strncmp(path, IGNORED_DIRS[i], len) == 0)
            if (path[len] == '/' || path[len] == '\0')
                return 1;
    }
    return 0;
}

static int is_critical(const char *path) {
    for (int i = 0; CRITICAL_FILES[i]; i++)
        if (strcmp(path, CRITICAL_FILES[i]) == 0)
            return 1;
    return 0;
}

int main(int argc, char *argv[])
{
    (void)argc; (void)argv;
    
    struct sigaction sa = {0};
    sa.sa_handler = signal_handler;
    sigaction(SIGINT, &sa, NULL);
    sigaction(SIGTERM, &sa, NULL);
    
    printf("=== Moniteur Fanotify ===\n\n");
    
    int fd = fanotify_init(
        FAN_CLASS_NOTIF | FAN_CLOEXEC | FAN_UNLIMITED_QUEUE,
        O_RDONLY | O_LARGEFILE | O_CLOEXEC
    );
    if (fd == -1) { perror("fanotify_init"); return 1; }
    
    if (fanotify_mark(fd, FAN_MARK_ADD | FAN_MARK_MOUNT,
                      FAN_MODIFY | FAN_CLOSE_WRITE,
                      AT_FDCWD, "/") == -1) {
        perror("fanotify_mark");
        close(fd);
        return 1;
    }
    
    printf("Surveillance active (Ctrl+C pour arrêter)\n\n");
    
    char *buffer = malloc(BUFFER_SIZE);
    if (!buffer) { perror("malloc"); close(fd); return 1; }
    
    unsigned long count = 0, critical = 0;
    
    while (g_running) {
        ssize_t len = read(fd, buffer, BUFFER_SIZE);
        if (len == -1) { if (errno == EINTR) continue; break; }
        
        struct fanotify_event_metadata *ev =
            (struct fanotify_event_metadata *)buffer;
        
        while (FAN_EVENT_OK(ev, len)) {
            if (ev->vers != FANOTIFY_METADATA_VERSION) break;
            if (ev->pid == getpid()) goto next;  /* Ignorer soi-même */
            
            char path[PATH_MAX] = "";
            if (ev->fd >= 0) get_file_path(ev->fd, path, sizeof(path));
            
            if (should_ignore(path)) goto next;
            
            count++;
            
            /* Colorer les fichiers critiques */
            if (is_critical(path)) {
                critical++;
                printf("\033[31m[CRITIQUE]\033[0m ");
            }
            
            char t[16];
            time_t now = time(NULL);
            strftime(t, sizeof(t), "%H:%M:%S", localtime(&now));
            
            printf("[%s] PID %-6d ", t, ev->pid);
            if (ev->mask & FAN_MODIFY) printf("MODIFY ");
            if (ev->mask & FAN_CLOSE_WRITE) printf("WRITE ");
            printf("%s\n", path);
            
        next:
            if (ev->fd >= 0) close(ev->fd);
            ev = FAN_EVENT_NEXT(ev, len);
        }
    }
    
    free(buffer);
    close(fd);
    
    printf("\n=== Statistiques ===\n");
    printf("Total: %lu | Critiques: %lu\n", count, critical);
    
    return 0;
}
```

### Compilation et test

```bash
# Compiler
gcc -Wall -Wextra -o moniteur moniteur.c

# Exécuter
sudo ./moniteur

# Dans un autre terminal, tester :
echo "test" > /tmp/test.txt
cat /etc/passwd
sudo nano /etc/hosts
```

---

# PARTIE 5 : Application au Projet HIDS

## Stratégie pour Sentinel

```
NIVEAU 1 : SURVEILLANCE GLOBALE (Mode Notification)
─────────────────────────────────────────────────────
• Marquer "/" avec FAN_MARK_MOUNT
• Événements : FAN_CLOSE_WRITE, FAN_MODIFY
• Objectif : Détecter toutes les modifications
• Action : Journaliser

NIVEAU 2 : FICHIERS CRITIQUES (Mode Permission)
─────────────────────────────────────────────────────
• Marquer chaque fichier critique individuellement
• Événements : FAN_OPEN_PERM, FAN_ACCESS_PERM
• Fichiers : /etc/passwd, /etc/shadow, /etc/sudoers
• Action : Vérifier whitelist avant d'autoriser

NIVEAU 3 : EXÉCUTION (Kernel 5.0+)
─────────────────────────────────────────────────────
• FAN_OPEN_EXEC_PERM pour contrôler les exécutions
• Vérifier les hashs, signatures, chemins autorisés
```

## Bonnes pratiques de sécurité

```c
/* 1. TOUJOURS VÉRIFIER LES RETOURS */
int fd = fanotify_init(flags, event_f_flags);
if (fd == -1) {
    perror("fanotify_init");
    exit(EXIT_FAILURE);
}

/* 2. UTILISER DES FONCTIONS SÉCURISÉES */
// MAUVAIS : sprintf(buffer, "Event: %s", path);
// BON :
snprintf(buffer, sizeof(buffer), "Event: %s", path);

/* 3. EFFACER LES DONNÉES SENSIBLES */
char password[64];
/* ... */
explicit_bzero(password, sizeof(password));

/* 4. RÉDUIRE LES PRIVILÈGES APRÈS INIT */
/* Après fanotify_init, on peut drop root */
```

---

# PARTIE 6 : Référence Rapide

```
┌─────────────────────────────────────────────────────────────────┐
│                 AIDE-MÉMOIRE FANOTIFY                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  INITIALISATION                                                 │
│  ─────────────────────────────────────────────────────────────  │
│  int fd = fanotify_init(flags, event_f_flags);                 │
│                                                                 │
│  FLAGS COURANTS:                                                │
│  • FAN_CLASS_NOTIF    - Mode notification                      │
│  • FAN_CLASS_CONTENT  - Mode permission                        │
│  • FAN_CLOEXEC        - Ferme sur exec()                       │
│  • FAN_NONBLOCK       - Non bloquant                           │
│                                                                 │
│  MARQUAGE                                                       │
│  ─────────────────────────────────────────────────────────────  │
│  fanotify_mark(fd, flags, mask, AT_FDCWD, path);               │
│                                                                 │
│  FLAGS:                                                         │
│  • FAN_MARK_ADD       - Ajoute                                 │
│  • FAN_MARK_REMOVE    - Retire                                 │
│  • FAN_MARK_MOUNT     - Point de montage                       │
│  • FAN_MARK_IGNORED_MASK - Whitelist                           │
│                                                                 │
│  ÉVÉNEMENTS:                                                    │
│  • FAN_OPEN, FAN_MODIFY, FAN_CLOSE_WRITE                       │
│  • FAN_OPEN_PERM, FAN_ACCESS_PERM (permission)                 │
│                                                                 │
│  LECTURE                                                        │
│  ─────────────────────────────────────────────────────────────  │
│  char buf[8192];                                               │
│  ssize_t len = read(fd, buf, sizeof(buf));                     │
│  struct fanotify_event_metadata *ev = (void*)buf;              │
│  while (FAN_EVENT_OK(ev, len)) {                               │
│      /* ev->mask, ev->fd, ev->pid */                           │
│      close(ev->fd);  /* NE PAS OUBLIER */                      │
│      ev = FAN_EVENT_NEXT(ev, len);                             │
│  }                                                              │
│                                                                 │
│  RÉPONSE PERMISSION                                             │
│  ─────────────────────────────────────────────────────────────  │
│  struct fanotify_response resp;                                │
│  resp.fd = ev->fd;                                             │
│  resp.response = FAN_ALLOW; /* ou FAN_DENY */                  │
│  write(fd, &resp, sizeof(resp));                               │
│                                                      │
│  CHEMIN DEPUIS FD                                    │
│  ────────────────────────────────────────────────────│
│  char path[PATH_MAX], p[32];                         │
│  snprintf(p, sizeof(p), "/proc/self/fd/%d", ev->fd); │
│  readlink(p, path, sizeof(path) - 1);                │
│                                                      │
│  COMPILATION                                         │
│  ────────────────────────────────────────────────────│
│  gcc -Wall -Wextra -o prog prog.c                    │
│  sudo ./prog                                         │
│                                                      │
└──────────────────────────────────────────────────────┘
```

## Codes d'erreur courants

| ERRNO    | Signification                              |
|----------|-------------------------------------------|
| EPERM    | Pas les privilèges (besoin root)          |
| ENOSYS   | fanotify non supporté par le kernel       |
| EINVAL   | Combinaison de flags invalide             |
| ENOMEM   | Plus de mémoire disponible                |
| ENOENT   | Fichier/chemin inexistant                 |
| EBADF    | Descripteur de fichier invalide           |

---

# Conclusion

Ce cours t'a fourni toutes les bases pour :

1. **Comprendre le C** : pointeurs, structures, allocation, errno
2. **Maîtriser fanotify** : init, mark, lecture, réponses
3. **Appliquer à Sentinel** : stratégie multi-niveaux

## Prochaines étapes

1. Phase 1 : Utilitaires de base (déjà commencé)
2. Phase 2 : Configuration INI
3. Phase 3 : Moteur fanotify (ce cours !)
4. Phase 4 : Règles et whitelists
5. Phase 5 : Alertes (fichier + webhook Discord)
6. Phase 6 : Daemonisation

## Ressources

- `man 7 fanotify` et `man 2 fanotify_init`
- https://docs.kernel.org/filesystems/fanotify.html
- Code kernel : `fs/notify/fanotify/`

---

*Cours créé pour le projet Sentinel HIDS*  
*Kernel cible : Linux 6.16+ (Kali Linux)*
