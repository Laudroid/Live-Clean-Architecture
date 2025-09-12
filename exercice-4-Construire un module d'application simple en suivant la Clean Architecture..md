
## TP : Construction d'un Module "Gestion de Livres" avec Clean Architecture

### Objectif du TP

Mettre en pratique les principes de la Clean Architecture en construisant un module d'application simple, en se concentrant sur la séparation des responsabilités entre les entités, les cas d'utilisation (use cases) et les adaptateurs d'interface.

### Énoncé du TP

Votre mission est de développer un module de gestion de livres. Ce module permettra d'effectuer des opérations de base sur des objets `Livre`.

**Fonctionnalités à implémenter :**

1.  **Ajouter un nouveau livre** : Permettre d'enregistrer un livre avec un titre, un auteur et un ISBN.
2.  **Lister tous les livres** : Afficher la liste de tous les livres enregistrés.
3.  **Afficher les détails d'un livre** : Récupérer et afficher les informations d'un livre spécifique via son identifiant.

**Détails de l'entité `Livre` :**

*   `id` (identifiant unique, généré)
*   `titre` (chaîne de caractères)
*   `auteur` (chaîne de caractères)
*   `isbn` (chaîne de caractères, unique)

### Architecture Cible

Vous allez structurer votre application en respectant les couches de la Clean Architecture. Voici une proposition de découpage pour ce module :

```
.
├── src
│   ├── domain             (Couche la plus interne : Entités et Règles Métier)
│   │   ├── entities
│   │   │   └── Book.js/ts/java/cs (ou Book.py)
│   │   └── boundaries     (Interfaces définissant les contrats pour les Use Cases et les Repositories)
│   │       ├── input      (Interfaces pour les Use Cases - ce que le monde extérieur peut demander)
│   │       │   ├── AddBookInputPort.js/ts/...
│   │       │   ├── ListBooksInputPort.js/ts/...
│   │       │   └── GetBookInputPort.js/ts/...
│   │       └── output     (Interfaces pour les Repositories et Presenters - ce dont les Use Cases ont besoin)
│   │           ├── BookRepositoryPort.js/ts/...
│   │           └── BookOutputPort.js/ts/... (pour la présentation des résultats)
│   ├── application        (Couche des Cas d'Utilisation / Interactors)
│   │   └── usecases
│   │       ├── AddBookUseCase.js/ts/...
│   │       ├── ListBooksUseCase.js/ts/...
│   │       └── GetBookUseCase.js/ts/...
│   ├── infrastructure     (Couche des Adaptateurs d'Interface et des Frameworks & Drivers)
│   │   ├── adapters
│   │   │   ├── controllers (Adaptateurs d'entrée : reçoivent les requêtes de l'UI/API)
│   │   │   │   └── BookController.js/ts/...
│   │   │   ├── presenters  (Adaptateurs de sortie : formatent les données pour l'UI/API)
│   │   │   │   └── BookConsolePresenter.js/ts/... (pour une sortie console simple)
│   │   │   └── repositories (Adaptateurs de persistance : implémentent BookRepositoryPort)
│   │   │       └── InMemoryBookRepositoryAdapter.js/ts/... (pour une persistance en mémoire simple)
│   │   └── main           (Point d'entrée de l'application)
│   │       └── app.js/ts/... (ou main.py, Program.cs, etc.)
```

### Tâches à Réaliser

1.  **Structure du Projet :**
    *   Créez une arborescence de dossiers conforme à la structure proposée ci-dessus.

2.  **Définition des Entités (`domain/entities`) :**
    *   Créez la classe `Book` avec les attributs `id`, `titre`, `auteur`, `isbn`.
    *   Assurez-vous que l'entité `Book` ne contient aucune logique spécifique à l'infrastructure (pas de méthodes de sauvegarde, de formatage UI, etc.). Elle représente uniquement les règles métier du livre.

3.  **Définition des Boundaries (`domain/ports`) :**
    *   **Input Ports (Interfaces pour les Use Cases) :**
        *   `AddBookInputPort` : Définit la méthode pour ajouter un livre.
        *   `ListBooksInputPort` : Définit la méthode pour lister les livres.
        *   `GetBookInputPort` : Définit la méthode pour obtenir un livre par ID.
    *   **Output Ports (Interfaces pour les Repositories et Presenters) :**
        *   `BookRepositoryPort` : Définit les méthodes nécessaires pour la persistance des livres (ex: `save(book)`, `findAll()`, `findById(id)`).
        *   `BookOutputPort` : Définit les méthodes pour présenter les résultats des cas d'utilisation (ex: `presentBooks(books)`, `presentBook(book)`, `presentError(error)`).

4.  **Implémentation des Cas d'Utilisation (`application/usecases`) :**
    *   Créez les classes `AddBookUseCase`, `ListBooksUseCase`, `GetBookUseCase`.
    *   Chaque Use Case doit implémenter son `Input Port` correspondant.
    *   Les Use Cases doivent orchestrer les opérations métier en utilisant le `BookRepositoryPort` (injecté via le constructeur) et le `BookOutputPort` pour communiquer les résultats.
    *   Ils ne doivent avoir aucune connaissance des détails de l'UI ou de la base de données.

5.  **Implémentation de la Persistance (`infrastructure/adapters/repositories`) :**
    *   Créez la classe `InMemoryBookRepositoryAdapter`.
    *   Cette classe doit implémenter le `BookRepositoryPort`.
    *   Utilisez une simple liste ou un tableau en mémoire pour stocker les livres.

6.  **Implémentation des Adaptateurs d'Interface (`infrastructure/adapters`) :**
    *   **Controller (`infrastructure/adapters/controllers`) :**
        *   Créez la classe `BookController`.
        *   Elle recevra les "requêtes" (par exemple, des appels de fonction dans une application console).
        *   Elle utilisera les `Input Ports` des Use Cases pour déclencher les opérations métier.
        *   Elle injectera les `Output Ports` (presenters) dans les Use Cases.
    *   **Presenter (`infrastructure/adapters/presenters`) :**
        *   Créez la classe `BookConsolePresenter`.
        *   Elle doit implémenter le `BookOutputPort`.
        *   Elle sera responsable de formater les données reçues des Use Cases pour un affichage console simple (ex: `console.log`).

7.  **Assemblage de l'Application (`infrastructure/main`) :**
    *   Dans votre fichier `app.js/ts/...`, instanciez toutes les dépendances (repository, use cases, controller, presenter).
    *   Injectez-les correctement les unes dans les autres pour construire le graphe d'objets de votre application.
    *   Simulez quelques interactions utilisateur (ex: ajouter 2-3 livres, puis lister-les, puis afficher un livre par son ID).

8.  **Tests (Optionnel mais recommandé) :**
    *   Écrivez quelques tests unitaires pour vos Use Cases. Vous devriez pouvoir les tester facilement en mockant le `BookRepositoryPort` et le `BookOutputPort`.

9.  **Implémentation d’un Frontend Simple (HTML/CSS/JS) :**
    *   Le but est de créer une interface web qui communique avec le contrôleur (`BookController`) et utilise les Use Cases déjà définis.
    *   Ce frontend ne doit contenir **aucune logique métier** : il se contente de capter les interactions utilisateur et d’afficher les résultats envoyés par le **presenter**.

    9.1. Structure

Ajoutez un dossier `frontend/` avec :

```
frontend/
 ├── index.html
 ├── styles.css
 └── app.js
```

    9.2. HTML (frontend/index.html)

Créez une page avec :

* Un formulaire pour ajouter un livre (`titre`, `auteur`, `isbn`).
* Un bouton pour lister les livres.
* Un champ pour rechercher un livre par ID.
* Une zone d’affichage des résultats.

    9.3. CSS minimaliste (frontend/styles.css)

    9.4. JS (frontend/app.js)

Ici, on fait office de **nouvel adaptateur d’interface**.
Au lieu de `console.log`, on met à jour le DOM.
Le frontend utilise le `BookController`, qui appelle les Use Cases, qui eux utilisent le `BookOutputPort`.
Pour cela, vous pouvez écrire un `BookDomPresenter` (dans `infrastructure/adapters/presenters`) qui implémente `BookOutputPort` et affiche dans le DOM.

    9.5. BookDomPresenter (infrastructure/adapters/presenters/BookDomPresenter.js)


### Consignes Spécifiques & Conseils

*   **Utilisation de l'IA :** Si vous le souhaitez, vous pouvez utiliser l'IA pour générer du code boilerplate (structures de classes, interfaces), explorer des syntaxes dans votre langage de prédilection, ou même pour obtenir des explications sur des concepts spécifiques de la Clean Architecture. Cependant, *votre compréhension* et *votre capacité à critiquer et adapter le code* sont primordiales. Ne copiez-collez pas sans comprendre.
*   **Langage de Programmation :** Vous êtes libre de choisir le langage de programmation de votre choix (JavaScript/TypeScript, Python, Java, C#, etc.). L'important est de respecter les principes architecturaux.
*   **Règle de Dépendance :** Rappelez-vous la règle d'or de la Clean Architecture : les dépendances ne peuvent pointer que vers l'intérieur. Les couches internes ne doivent jamais dépendre des couches externes. Utilisez l'injection de dépendances pour respecter ce principe.
*   **Testabilité :** Remarquez comment la séparation des préoccupations rend chaque couche, et en particulier les cas d'utilisation, beaucoup plus facile à tester de manière isolée.
*   **Progression :** Commencez simple. Implémentez une fonctionnalité à la fois (par exemple, l'ajout de livre), puis ajoutez les autres.

### Livrables

*   Un dépôt Git contenant le code source de votre appli, respectant la structure de la Clean Architecture.
*   Un fichier `README.md` expliquant 
*   *comment exécuter votre application
*   *vos réflexions sur les défis rencontrés
*   *les bénéfices perçus des choix que vous avez effectué.

---

Bon courage et amusez-vous bien à construire ce module !