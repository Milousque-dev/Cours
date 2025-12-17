# Chapitre 8 : Structures, unions et énumérations

## Table des matières

1.  [Structures (struct)](#structures-struct)
2.  [Accès aux membres](#accès-aux-membres)
3.  [Structures et fonctions](#structures-et-fonctions)
4.  [Structures imbriquées](#structures-imbriquées)
5.  [Tableaux de structures](#tableaux-de-structures)
6.  [typedef](#typedef)
7.  [Unions](#unions)
8.  [Énumérations (enum)](#énumérations-enum)
9.  [Champs de bits](#champs-de-bits)
10. [Alignement et padding](#alignement-et-padding)
11. [Comparaison avec Python/JS/C#/Go](#comparaison-avec-pythonjscgo)
12. [Exercices du chapitre](#exercices-du-chapitre)

---

## Structures (struct)

### Définition

Une **structure** regroupe plusieurs variables de types différents sous un même nom :

```c
#include <stdio.h>

// Définition d'une structure
struct Personne {
    char nom[50];
    char prenom[50];
    int age;
    float taille;
};

int main(void) {
    // Déclaration d'une variable de type struct Personne
    struct Personne p1;
    
    // Initialisation des membres
    strcpy(p1.nom, "Dupont");
    strcpy(p1.prenom, "Jean");
    p1.age = 30;
    p1.taille = 1.75;
    
    // Déclaration avec initialisation
    struct Personne p2 = {"Martin", "Marie", 25, 1.65};
    
    // Initialisation désignée (C99+)
    struct Personne p3 = {
        .prenom = "Pierre",
        .nom = "Durand",
        .age = 40,
        .taille = 1.80
    };
    
    // Initialisation partielle (reste à 0)
    struct Personne p4 = {.nom = "Test"};
    
    printf("%s %s, %d ans, %.2fm\n", 
           p2.prenom, p2.nom, p2.age, p2.taille);
    
    return 0;
}
```

### Syntaxes de déclaration

```c
// Méthode 1 : Structure nommée, variables séparées
struct Point {
    int x;
    int y;
};
struct Point p1, p2;

// Méthode 2 : Structure et variables en même temps
struct Rectangle {
    int largeur;
    int hauteur;
} rect1, rect2;

// Méthode 3 : Structure anonyme (peu recommandé)
struct {
    int a;
    int b;
} variable_unique;

// Méthode 4 : typedef (recommandé)
typedef struct {
    int x;
    int y;
} Point;

Point p3;  // Pas besoin de "struct" devant
```

---

## Accès aux membres

### Opérateur point `.`

Pour accéder aux membres d'une structure :

```c
struct Point {
    int x;
    int y;
};

struct Point p = {10, 20};
printf("x = %d, y = %d\n", p.x, p.y);

p.x = 30;
p.y = 40;
```

### Opérateur flèche `->`

Pour accéder aux membres via un **pointeur** vers structure :

```c
struct Point p = {10, 20};
struct Point *ptr = &p;

// Méthode 1 : déréférencement puis point
printf("x = %d\n", (*ptr).x);

// Méthode 2 : opérateur flèche (plus lisible)
printf("x = %d\n", ptr->x);  // Équivalent à (*ptr).x

// Modification via pointeur
ptr->x = 100;
ptr->y = 200;
```

---

## Structures et fonctions

### Passage par valeur (copie)

```c
#include <stdio.h>
#include <string.h>

typedef struct {
    char nom[50];
    int age;
} Personne;

// Passage par valeur : reçoit une COPIE
void afficher_personne(Personne p) {
    printf("%s, %d ans\n", p.nom, p.age);
    p.age = 0;  // Modifie seulement la copie
}

int main(void) {
    Personne p = {"Alice", 30};
    afficher_personne(p);
    printf("Âge après : %d\n", p.age);  // Toujours 30
    return 0;
}
```

### Passage par référence (pointeur) - recommandé

```c
// Passage par pointeur : plus efficace, peut modifier
void vieillir(Personne *p) {
    p->age++;  // Modifie l'original
}

// const pour lecture seule
void afficher(const Personne *p) {
    printf("%s, %d ans\n", p->nom, p->age);
    // p->age = 0;  // ERREUR : const interdit modification
}

int main(void) {
    Personne p = {"Bob", 25};
    vieillir(&p);
    afficher(&p);  // Bob, 26 ans
    return 0;
}
```

### Retourner une structure

```c
typedef struct {
    int x;
    int y;
} Point;

// Retour par valeur (copie)
Point creer_point(int x, int y) {
    Point p = {x, y};
    return p;  // Retourne une copie
}

// Ou plus compact
Point creer_point2(int x, int y) {
    return (Point){x, y};  // Littéral composé (C99+)
}

int main(void) {
    Point p = creer_point(10, 20);
    printf("(%d, %d)\n", p.x, p.y);
    return 0;
}
```

---

## Structures imbriquées

```c
#include <stdio.h>

typedef struct {
    int jour;
    int mois;
    int annee;
} Date;

typedef struct {
    int numero;
    char rue[100];
    char ville[50];
    int code_postal;
} Adresse;

typedef struct {
    char nom[50];
    char prenom[50];
    Date naissance;      // Structure imbriquée
    Adresse adresse;     // Structure imbriquée
} Personne;

int main(void) {
    Personne p = {
        .nom = "Dupont",
        .prenom = "Jean",
        .naissance = {15, 3, 1990},
        .adresse = {
            .numero = 42,
            .rue = "Rue de la Paix",
            .ville = "Paris",
            .code_postal = 75001
        }
    };
    
    // Accès aux membres imbriqués
    printf("%s %s\n", p.prenom, p.nom);
    printf("Né le %d/%d/%d\n", 
           p.naissance.jour, p.naissance.mois, p.naissance.annee);
    printf("Habite au %d %s, %d %s\n",
           p.adresse.numero, p.adresse.rue,
           p.adresse.code_postal, p.adresse.ville);
    
    return 0;
}
```

---

## Tableaux de structures

```c
#include <stdio.h>
#include <string.h>

typedef struct {
    char nom[50];
    float note;
} Etudiant;

#define MAX_ETUDIANTS 100

int main(void) {
    // Tableau de structures
    Etudiant classe[MAX_ETUDIANTS];
    int nb_etudiants = 0;
    
    // Initialisation
    Etudiant promo[] = {
        {"Alice", 15.5},
        {"Bob", 12.0},
        {"Charlie", 18.0},
        {"Diana", 14.5}
    };
    int n = sizeof(promo) / sizeof(promo[0]);
    
    // Parcours
    float somme = 0;
    for (int i = 0; i < n; i++) {
        printf("%s : %.1f\n", promo[i].nom, promo[i].note);
        somme += promo[i].note;
    }
    printf("Moyenne : %.2f\n", somme / n);
    
    // Recherche
    char recherche[] = "Bob";
    for (int i = 0; i < n; i++) {
        if (strcmp(promo[i].nom, recherche) == 0) {
            printf("Trouvé : %s avec %.1f\n", promo[i].nom, promo[i].note);
            break;
        }
    }
    
    // Tri par note (bubble sort simple)
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (promo[j].note < promo[j + 1].note) {
                Etudiant temp = promo[j];
                promo[j] = promo[j + 1];
                promo[j + 1] = temp;
            }
        }
    }
    
    printf("\nClassement :\n");
    for (int i = 0; i < n; i++) {
        printf("%d. %s : %.1f\n", i + 1, promo[i].nom, promo[i].note);
    }
    
    return 0;
}
```

---

## typedef

`typedef` crée un alias pour un type existant :

```c
// Types simples
typedef unsigned char byte;
typedef unsigned int uint;
typedef unsigned long ulong;

// Pointeurs
typedef char* string;
typedef int* IntPtr;

// Structures
typedef struct {
    int x;
    int y;
} Point;

// Structure avec référence à elle-même (liste chaînée)
typedef struct Node {
    int data;
    struct Node *next;  // Doit utiliser "struct Node", pas "Node"
} Node;

// Pointeur de fonction
typedef int (*Comparateur)(const void*, const void*);

// Utilisation
byte b = 255;
Point p = {10, 20};
Node *liste = NULL;
Comparateur cmp = strcmp;
```

---

## Unions

Une **union** partage le même espace mémoire pour tous ses membres. Un seul membre est valide à la fois :

```c
#include <stdio.h>

union Data {
    int entier;
    float flottant;
    char caractere;
};

int main(void) {
    union Data d;
    
    printf("sizeof(union Data) = %zu\n", sizeof(d));  // Taille du plus grand membre
    
    d.entier = 42;
    printf("entier : %d\n", d.entier);
    
    d.flottant = 3.14;  // Écrase entier!
    printf("flottant : %f\n", d.flottant);
    printf("entier (corrompu) : %d\n", d.entier);  // Valeur imprévisible
    
    return 0;
}
```

### Cas d'usage : Tagged union (union discriminée)

```c
#include <stdio.h>

typedef enum {
    TYPE_INT,
    TYPE_FLOAT,
    TYPE_STRING
} TypeTag;

typedef struct {
    TypeTag type;  // Le "tag" qui indique quel membre est actif
    union {
        int valeur_int;
        float valeur_float;
        char valeur_string[50];
    } data;
} Variant;

void afficher_variant(Variant *v) {
    switch (v->type) {
        case TYPE_INT:
            printf("Int: %d\n", v->data.valeur_int);
            break;
        case TYPE_FLOAT:
            printf("Float: %f\n", v->data.valeur_float);
            break;
        case TYPE_STRING:
            printf("String: %s\n", v->data.valeur_string);
            break;
    }
}

int main(void) {
    Variant v1 = {TYPE_INT, .data.valeur_int = 42};
    Variant v2 = {TYPE_FLOAT, .data.valeur_float = 3.14};
    Variant v3 = {TYPE_STRING};
    strcpy(v3.data.valeur_string, "Hello");
    
    afficher_variant(&v1);
    afficher_variant(&v2);
    afficher_variant(&v3);
    
    return 0;
}
```

### Type punning (réinterprétation de bits)

```c
#include <stdio.h>

union FloatInt {
    float f;
    unsigned int i;
};

int main(void) {
    union FloatInt x;
    x.f = 3.14159f;
    
    printf("Float: %f\n", x.f);
    printf("Bits (hex): 0x%08X\n", x.i);
    
    // Voir la représentation IEEE 754
    // 3.14159 ≈ 0x40490FD0
    
    return 0;
}
```

---

## Énumérations (enum)

Les **énumérations** définissent un ensemble de constantes entières nommées :

```c
#include <stdio.h>

// Énumération simple
enum Couleur {
    ROUGE,      // 0
    VERT,       // 1
    BLEU        // 2
};

// Avec valeurs explicites
enum Mois {
    JANVIER = 1,
    FEVRIER,    // 2
    MARS,       // 3
    // ...
    DECEMBRE = 12
};

// Flags avec puissances de 2
enum Permissions {
    PERM_NONE    = 0,
    PERM_READ    = 1,   // 0001
    PERM_WRITE   = 2,   // 0010
    PERM_EXECUTE = 4,   // 0100
    PERM_ALL     = 7    // 0111
};

int main(void) {
    enum Couleur c = VERT;
    printf("Couleur : %d\n", c);  // 1
    
    // switch avec enum
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
    
    // Combinaison de flags avec |
    int perms = PERM_READ | PERM_WRITE;
    
    // Test de flag avec &
    if (perms & PERM_READ) {
        printf("Lecture autorisée\n");
    }
    if (perms & PERM_EXECUTE) {
        printf("Exécution autorisée\n");  // Non affiché
    }
    
    return 0;
}
```

---

## Champs de bits

Permettent de spécifier la taille exacte en bits de chaque membre :

```c
#include <stdio.h>

// Structure avec champs de bits
struct Registre {
    unsigned int actif : 1;      // 1 bit
    unsigned int priorite : 3;   // 3 bits (0-7)
    unsigned int type : 4;       // 4 bits (0-15)
    unsigned int code : 8;       // 8 bits (0-255)
    unsigned int reserve : 16;   // 16 bits
};  // Total : 32 bits = 4 octets

int main(void) {
    struct Registre reg = {0};
    
    reg.actif = 1;
    reg.priorite = 5;
    reg.type = 10;
    reg.code = 200;
    
    printf("Actif: %u\n", reg.actif);
    printf("Priorité: %u\n", reg.priorite);
    printf("Type: %u\n", reg.type);
    printf("Code: %u\n", reg.code);
    printf("sizeof: %zu\n", sizeof(reg));
    
    // Attention : débordement silencieux
    reg.priorite = 15;  // Seuls 3 bits : stocke 15 & 0x7 = 7
    printf("Priorité après 15: %u\n", reg.priorite);
    
    return 0;
}
```

---

## Alignement et padding

Le compilateur ajoute du **padding** pour aligner les membres sur des adresses optimales :

```c
#include <stdio.h>

struct Exemple1 {
    char a;      // 1 octet
    int b;       // 4 octets
    char c;      // 1 octet
};  // Taille réelle : 12 octets (pas 6!)

struct Exemple2 {
    int b;       // 4 octets
    char a;      // 1 octet
    char c;      // 1 octet
};  // Taille réelle : 8 octets

int main(void) {
    printf("sizeof(Exemple1) = %zu\n", sizeof(struct Exemple1));  // 12
    printf("sizeof(Exemple2) = %zu\n", sizeof(struct Exemple2));  // 8
    
    // Visualisation du padding
    struct Exemple1 e1;
    printf("Adresse de a: %p\n", (void*)&e1.a);
    printf("Adresse de b: %p\n", (void*)&e1.b);
    printf("Adresse de c: %p\n", (void*)&e1.c);
    
    return 0;
}
```

### Règle d'or : Ordonner les membres du plus grand au plus petit

```c
// MAUVAIS : beaucoup de padding
struct Inefficace {
    char a;      // 1 + 7 padding
    double b;    // 8
    char c;      // 1 + 7 padding
};  // 24 octets

// BON : minimal padding
struct Efficace {
    double b;    // 8
    char a;      // 1
    char c;      // 1 + 6 padding
};  // 16 octets
```

---

## Comparaison avec Python/JS/C#/Go

| Aspect           | C struct | Python class | JS object  | C# class/struct       | Go struct  |
|------------------|----------|--------------|------------|-----------------------|------------|
| Héritage         | Non      | Oui          | Prototypes | Oui (class)           | Non        |
| Méthodes         | Non      | Oui          | Oui        | Oui                   | Oui        |
| Encapsulation    | Non      | Convention   | Convention | Oui                   | Convention |
| Valeur/Référence | Valeur   | Référence    | Référence  | struct=val, class=ref | Valeur     |

```go
// Go : struct similaire au C mais avec méthodes
type Point struct {
    X int
    Y int
}

func (p Point) Distance() float64 {
    return math.Sqrt(float64(p.X*p.X + p.Y*p.Y))
}
```

```python
# Python : classes avec méthodes intégrées
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def distance(self):
        return (self.x**2 + self.y**2)**0.5
```

---

## Exercices du chapitre

### Exercice 8.1 : Structure simple (Débutant)
Créez une structure `Livre` avec titre, auteur, année, prix et des fonctions pour :
- Initialiser un livre
- Afficher un livre
- Comparer deux livres par année

### Exercice 8.2 : Tableau de structures (Intermédiaire)
Créez un carnet d'adresses avec une structure `Contact` et implémentez :
- Ajout de contact
- Recherche par nom
- Tri par nom
- Sauvegarde/chargement depuis fichier

### Exercice 8.3 : Structures imbriquées (Intermédiaire)
Créez un système de gestion d'employés avec :
- `Date` (jour, mois, année)
- `Adresse` (rue, ville, code postal)
- `Employe` (nom, date_naissance, date_embauche, adresse, salaire)

### Exercice 8.4 : Union discriminée (Intermédiaire-Avancé)
Implémentez un type `Valeur` qui peut contenir soit un int, soit un float, soit une chaîne, avec les fonctions appropriées.

### Exercice 8.5 : Liste chaînée (Avancé)
Implémentez une liste chaînée générique de structures :
- Insertion (début, fin, position)
- Suppression
- Recherche
- Tri

---

## Résumé du chapitre

**struct** : Regroupe des variables de types différents

**Accès** : `.` pour variable, `->` pour pointeur

**typedef** : Crée des alias de types pour plus de clarté

**union** : Partage la mémoire, un seul membre actif à la fois

**enum** : Constantes entières nommées

**Champs de bits** : Contrôle précis de la taille des membres

**Padding** : Ordonnez les membres pour minimiser la taille

---

**Prochain chapitre** : Gestion de la mémoire et fichiers
