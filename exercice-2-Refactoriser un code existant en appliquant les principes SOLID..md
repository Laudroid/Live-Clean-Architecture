Ce TP vise à vous immerger dans la pratique du refactoring et l'application des principes SOLID. Vous travaillerez sur un code existant, identifierez ses faiblesses et le transformerez pour le rendre plus robuste, maintenable et évolutif.

---

## TP : Refactoring SOLID d'un Système de Gestion de Produits

### Contexte

Vous disposez d'un petit système de gestion de produits. Actuellement, il permet de créer, consulter, modifier et supprimer des produits. Le code, bien que fonctionnel, a été développé rapidement et présente des signes de rigidité et de fragilité, rendant toute évolution complexe.

### Objectif du TP

L'objectif est de refactoriser ce code pour qu'il respecte les principes SOLID. Vous devrez identifier les violations existantes, appliquer les principes pour les corriger, puis implémenter une nouvelle fonctionnalité en vous appuyant sur votre code refactorisé.

### Prérequis

*   Connaissance de base en programmation orientée objet (classes, interfaces, héritage).
*   Compréhension des principes SOLID.
*   Maîtrise d'un langage de programmation orienté objet (C#, Java, Python, TypeScript, etc.).
*   Un environnement de développement (IDE) de votre choix.
*   Accès à un outil de gestion de versions (Git recommandé).

### Le Code Existant (Point de Départ)

Le codebase initial (fourni via un dépôt Git, par exemple : `https://github.com/votre-organisation/tp-solid-product-manager-starter`) contient les éléments suivants :

*   Une classe `Product` (ou équivalent) : Représente un produit avec des propriétés comme `Id`, `Name`, `Description`, `Price`, `Category`.
*   Une classe `ProductService` (ou équivalent) : C'est le cœur monolithique de l'application. Elle gère :
    *   Les opérations CRUD (Create, Read, Update, Delete) pour les produits.
    *   La validation des données des produits.
    *   L'interaction directe avec un mécanisme de persistance (par exemple, une liste en mémoire, un fichier CSV, ou des appels directs à une base de données simplifiée).
*   Une classe `ProductRepository` (ou équivalent) : Gère la persistance des produits, souvent directement couplée au `ProductService`.

**Observations initiales probables (sans être exhaustif) :**

*   Le `ProductService` est volumineux et contient de nombreuses responsabilités.
*   Les dépendances sont souvent directes (par exemple, `ProductService` instancie directement `ProductRepository`).
*   L'ajout d'un nouveau type de produit ou d'une nouvelle règle de validation pourrait nécessiter des modifications dans plusieurs endroits du `ProductService`.

### Les Étapes du TP

#### Étape 1 : Analyse et Identification des Violations

1.  **Exploration du Code :** Parcourez le code fourni. Comprenez son fonctionnement actuel et identifiez les différentes responsabilités assumées par chaque classe.
2.  **Identification des Violations SOLID :**
    *   Pour chaque principe SOLID (SRP, OCP, LSP, ISP, DIP), identifiez les endroits où le code actuel ne le respecte pas.
    *   Documentez brièvement vos observations : quelle classe viole quel principe et pourquoi.
    *   **Conseil IA :** N'hésitez pas à utiliser des outils d'IA (comme ChatGPT, Copilot, etc.) pour vous aider à identifier des "code smells" ou des suggestions de refactoring. Demandez-lui d'analyser des extraits de code spécifiques et de pointer les violations SOLID potentielles. Soyez critique face à ses propositions et comprenez le "pourquoi" derrière chaque suggestion.

#### Étape 2 : Application du Principe de Responsabilité Unique (SRP)

1.  **Décomposition du `ProductService` :** Le `ProductService` est probablement le candidat principal pour le SRP.
    *   Séparez les responsabilités : création, lecture, mise à jour, suppression, validation, etc.
    *   Créez de nouvelles classes ou modules, chacun avec une seule raison de changer. Par exemple :
        *   `ProductValidator` (ou une interface `IProductValidator`).
        *   `ProductCreator` (ou `IProductCreator`).
        *   `ProductRetriever` (ou `IProductRetriever`).
        *   `ProductUpdater` (ou `IProductUpdater`).
        *   `ProductDeleter` (ou `IProductDeleter`).
    *   Assurez-vous que chaque nouvelle entité a une responsabilité clairement définie et unique.

#### Étape 3 : Application du Principe Ouvert/Fermé (OCP) et d'Inversion des Dépendances (DIP)

1.  **Introduction d'Abstractions pour la Persistance :**
    *   Créez une interface pour le `ProductRepository` (par exemple, `IProductRepository`). Cette interface définira les opérations de persistance (ex: `Add`, `GetById`, `GetAll`, `Update`, `Delete`).
    *   Modifiez la classe `ProductRepository` existante pour qu'elle implémente cette interface.
    *   Modifiez les classes qui dépendent de la persistance (vos nouveaux `ProductCreator`, `ProductRetriever`, etc.) pour qu'elles dépendent de l'interface `IProductRepository` et non de l'implémentation concrète. Utilisez l'injection de dépendances (par constructeur, par exemple).
2.  **Gestion des Types de Produits (OCP) :**
    *   Si votre système doit gérer différents types de produits (ex: `PhysicalProduct`, `DigitalProduct`) avec des logiques spécifiques, réfléchissez à comment l'ajouter sans modifier les classes existantes.
    *   Cela pourrait impliquer l'utilisation de stratégies ou de fabriques. Pour ce TP, si ce n'est pas directement applicable, assurez-vous au moins que l'ajout d'une nouvelle *règle de validation* ou d'une nouvelle *source de données* ne casse pas l'OCP.

#### Étape 4 : Application du Principe de Ségrégation des Interfaces (ISP) et de Substitution de Liskov (LSP) (si pertinent)

1.  **Examen des Interfaces (ISP) :**
    *   Examinez les interfaces que vous avez créées (ex: `IProductRepository`, `IProductValidator`). Sont-elles trop "grasses" ?
    *   Si une interface contient des méthodes que tous ses clients n'utilisent pas, séparez-la en interfaces plus petites et plus spécifiques. Par exemple, `IProductReader` et `IProductWriter` au lieu d'une seule `IProductRepository` pour certains contextes.
2.  **Vérification de la Substitution de Liskov (LSP) :**
    *   Si vous avez introduit des hiérarchies de classes (par exemple, différents types de `ProductValidator` ou de `Product`), assurez-vous que les sous-types peuvent être substitués à leurs types de base sans altérer la justesse du programme.
    *   Bien que moins fréquent de "violer" LSP dans un code initial simple, c'est une bonne pratique de vérifier que vos nouvelles abstractions respectent cette règle.

#### Étape 5 : Implémentation d'une Nouvelle Fonctionnalité

1.  **Ajout d'une Fonctionnalité :** Implémentez la fonctionnalité suivante :
    *   **Recherche de produits par catégorie et par prix maximum.**
    *   Cette fonctionnalité doit être implémentée en respectant scrupuleusement les principes SOLID que vous venez d'appliquer. Elle devrait idéalement nécessiter peu ou pas de modifications dans les classes existantes, mais plutôt l'ajout de nouvelles classes ou l'extension d'interfaces existantes.
2.  **Validation :** Vérifiez que l'ajout de cette fonctionnalité n'a pas réintroduit de violations SOLID et que votre code reste propre et extensible.

### Consignes Générales et Conseils

*   **Travaillez par petites itérations :** Ne tentez pas de tout refactoriser d'un coup. Concentrez-vous sur un principe ou une responsabilité à la fois.
*   **Tests :** Même si ce n'est pas l'objectif principal du TP, la mise en place de quelques tests unitaires simples (avant et après refactoring) peut vous aider à valider que votre code fonctionne toujours comme prévu.
*   **L'IA est un outil, pas un remplaçant :** Utilisez l'IA pour générer des idées, des ébauches de code, ou identifier des problèmes. Cependant, la compréhension, l'analyse critique et la décision finale vous appartiennent. Ne copiez-collez pas aveuglément.
*   **Documentation :** Documentez brièvement vos choix de refactoring (par exemple, dans les commentaires de code ou un petit fichier README). Expliquez pourquoi vous avez appliqué tel principe à tel endroit.
*   **Versionnement :** Utilisez Git. Faites des commits réguliers avec des messages clairs, surtout après chaque étape majeure de refactoring.

### Rendu

Le rendu attendu est un dépôt Git contenant :

1.  Le code refactorisé.
2.  L'implémentation de la nouvelle fonctionnalité.
3.  Un fichier `README.md` à la racine du dépôt décrivant :
    *   Les violations SOLID initiales que vous avez identifiées.
    *   Les principales modifications apportées pour respecter chaque principe SOLID.
    *   Comment la nouvelle fonctionnalité a été implémentée en respectant les principes SOLID.

### Évaluation

Votre travail sera évalué sur :

*   La pertinence de l'identification des violations SOLID initiales.
*   La qualité du refactoring et le respect des principes SOLID.
*   La propreté et la lisibilité du code final.
*   La bonne implémentation de la nouvelle fonctionnalité.
*   La clarté et la complétude de votre documentation dans le `README.md`.

Bon courage !