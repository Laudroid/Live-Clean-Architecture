
## TP : Tests et Stratégies de Déploiement pour une Application Clean Architecture

### Contexte

Vous disposez d'une structure de projet avec des Entités, des Cas d'Utilisation, des Ports et des Adaptateurs initiaux. L'objectif est maintenant de garantir la robustesse de cette application par des tests et d'envisager son déploiement.

### Objectif du TP

*   Implémenter des tests unitaires pour les composants du domaine (Entités, Cas d'Utilisation).
*   Écrire des tests d'intégration pour les adaptateurs.
*   Analyser et discuter des stratégies de déploiement adaptées à cette architecture.

### Prérequis

*   Projet fonctionnel (même avec des implémentations minimales).
*   Connaissance des bases de votre langage de programmation et de son framework de test unitaire (ex: JUnit/Mockito pour Java, Pytest/unittest.mock pour Python, Jest/Mocha pour JS/TS, XUnit/NSubstitute pour C#).

### Mini-Projet : Gestion d'un Catalogue de Produits

Pour ce TP, nous allons considérer que votre application gère un catalogue de produits.

**Rappel des Composants Clés :**

*   **Entité : `Product`**
    *   Attributs : `id` (UUID/String), `name` (String), `description` (String), `price` (BigDecimal/Float), `stock` (Integer).
    *   Règles métier :
        *   Le `name` ne peut pas être vide.
        *   Le `price` doit être strictement positif.
        *   Le `stock` doit être supérieur ou égal à zéro.
    *   Méthodes : `updateName(newName)`, `updateDescription(newDescription)`, `changePrice(newPrice)`, `adjustStock(quantity)`.

*   **Cas d'Utilisation (Use Cases) :**
    *   `CreateProductUseCase` : Crée un nouveau produit.
    *   `GetProductByIdUseCase` : Récupère un produit par son ID.
    *   `UpdateProductUseCase` : Met à jour les informations d'un produit existant.
    *   `DeleteProductUseCase` : Supprime un produit.
    *   `GetAllProductsUseCase` : Récupère tous les produits.

*   **Ports (Interfaces) :**
    *   `ProductRepository` : Interface définissant les opérations de persistance (ex: `save(Product)`, `findById(id)`, `update(Product)`, `delete(id)`, `findAll()`).
    *   `ProductOutputPort` : Interface pour la présentation des résultats (ex: `presentProduct(Product)`, `presentError(Error)`).

*   **Adaptateurs (Initial) :**
    *   `InMemoryProductRepository` : Une implémentation simple de `ProductRepository` utilisant une collection en mémoire pour stocker les produits.

---

### Partie 1 : Tests Unitaires (Core Domain)

Cette partie vise à tester le cœur de votre application, indépendant de toute technologie externe.

**1.1. Tests des Entités (`Product`)**

Écrivez des tests unitaires pour l'entité `Product`.

*   Vérifiez que les règles métier sont correctement appliquées lors de la création et de la modification d'un produit.
    *   Exemple : Tentez de créer un produit avec un nom vide, un prix négatif ou un stock négatif et assurez-vous que les erreurs appropriées sont levées (ou que l'état est invalide).
*   Testez les méthodes de l'entité (`updateName`, `changePrice`, `adjustStock`) pour confirmer qu'elles modifient l'état de l'entité comme prévu et respectent les invariants.
    *   Exemple : Après avoir ajusté le stock, vérifiez que la nouvelle valeur est correcte.

**1.2. Tests des Cas d'Utilisation (Use Cases)**

Écrivez des tests unitaires pour chaque Cas d'Utilisation.

*   **Isolation :** Chaque Use Case doit être testé isolément. Utilisez des *mocks* ou des *stubs* pour simuler le comportement de ses dépendances (ex: `ProductRepository`, `ProductOutputPort`).
*   **Scénarios Nominaux :** Testez les chemins de succès pour chaque Use Case.
    *   Exemple pour `CreateProductUseCase` : Vérifiez qu'un produit est créé avec succès et que `productRepository.save()` est appelé avec les bonnes données.
    *   Exemple pour `GetProductByIdUseCase` : Vérifiez qu'un produit existant est correctement récupéré.
*   **Scénarios d'Erreur :** Testez les cas où des erreurs sont attendues.
    *   Exemple pour `GetProductByIdUseCase` : Simulez un `productRepository.findById()` qui retourne `null` (produit non trouvé) et vérifiez que le Use Case gère cette situation (ex: lève une exception spécifique, appelle `productOutputPort.presentError()`).
    *   Exemple pour `UpdateProductUseCase` : Testez la mise à jour d'un produit inexistant ou avec des données invalides.
*   **Vérification des Interactions :** Assurez-vous que les Use Cases interagissent correctement avec leurs ports (ex: `productRepository.save()` est appelé une fois, `productOutputPort.presentProduct()` est appelé avec le bon objet).

### Partie 2 : Tests d'Intégration (Adapters)

Cette partie se concentre sur la vérification de l'interaction entre les composants, notamment les adaptateurs et les infrastructures externes.

**2.1. Tests de l'Adaptateur `InMemoryProductRepository`**

Écrivez des tests d'intégration pour l'implémentation `InMemoryProductRepository`.

*   Confirmez que cet adaptateur respecte le contrat de l'interface `ProductRepository`.
*   Testez les opérations CRUD (Create, Read, Update, Delete) de manière intégrée :
    *   Créez un produit, puis tentez de le récupérer par son ID.
    *   Créez plusieurs produits, puis vérifiez que `findAll()` les retourne tous.
    *   Mettez à jour un produit existant et vérifiez que les modifications sont persistées.
    *   Supprimez un produit et assurez-vous qu'il n'est plus récupérable.
*   Assurez-vous que les cas limites sont gérés (ex: tentative de récupérer un produit inexistant, suppression d'un produit déjà supprimé).

**2.2. (Optionnel mais recommandé) Tests avec une base de données réelle**

Si vous avez déjà une implémentation de `ProductRepository` utilisant une base de données réelle (ex: PostgreSQL, MySQL, MongoDB), écrivez des tests d'intégration pour celle-ci.

*   Utilisez des outils comme Testcontainers (pour Java), Docker Compose ou des bases de données en mémoire (H2 pour Java, SQLite pour Python) pour démarrer une instance de base de données dédiée aux tests.
*   Chaque test doit démarrer avec un état propre de la base de données pour garantir l'indépendance et la reproductibilité.
*   Appliquez les mêmes principes de test CRUD que pour l'`InMemoryProductRepository`, mais en vérifiant l'interaction avec la base de données réelle.

### Partie 3 : Discussion sur les Stratégies de Déploiement

Rédigez une courte section (1-2 pages) répondant aux questions suivantes, en vous basant sur l'architecture de votre application et les principes de la Clean Architecture.

*   **Impact de la Clean Architecture sur le Déploiement :**
    *   Comment la séparation des préoccupations et la dépendance inversée facilitent-elles ou contraignent-elles les choix de déploiement ?
    *   En quoi le fait que le domaine ne dépende pas de l'infrastructure est-il un avantage pour le déploiement ?
*   **Options de Déploiement :**
    *   Quelles sont les options de déploiement envisageables pour cette application (ex: monolithe, microservices, serverless) ?
    *   Pour chacune de ces options, décrivez brièvement comment votre application pourrait être déployée.
    *   Discutez des avantages et inconvénients de chaque stratégie spécifiquement pour notre mini-projet de gestion de catalogue de produits.
*   **Rôle des Conteneurs et de l'Orchestration :**
    *   Quel rôle jouent les conteneurs (Docker) dans le déploiement d'une application suivant la Clean Architecture ?
    *   Comment l'orchestration (Kubernetes, Docker Swarm) peut-elle être utilisée pour gérer le déploiement et la scalabilité de cette application ?
*   **Intégration Continue / Déploiement Continu (CI/CD) :**
    *   Comment une chaîne CI/CD pourrait-elle être mise en place pour automatiser les tests (unitaires et d'intégration) et le déploiement de cette application ?
    *   Quelles étapes clés inclurait cette chaîne ?

### Consignes Générales & Conseils

*   **Qualité du Code de Test :** Vos tests doivent être clairs, lisibles et bien organisés. Suivez les principes F.I.R.S.T. (Fast, Independent, Repeatable, Self-validating, Timely).
*   **Couverture de Code :** Visez une bonne couverture de code, en vous assurant que les chemins critiques et les cas d'erreur sont testés.
*   **Utilisation de l'IA :** L'IA peut être un excellent assistant pour générer des squelettes de tests, suggérer des cas de test ou aider à la syntaxe spécifique de votre framework de test. Cependant, le but de ce TP est de renforcer votre compréhension.
    *   **Comprenez :** Ne copiez-collez pas sans comprendre le code généré.
    *   **Validez :** Vérifiez que les tests générés correspondent bien aux exigences de votre application et à la logique que vous souhaitez tester.
    *   **Adaptez :** L'IA peut ne pas toujours générer le code le plus idiomatique ou le plus optimisé pour votre contexte. Adaptez-le si nécessaire.
    *   **Réfléchissez :** Utilisez l'IA comme un outil pour accélérer le processus, mais la réflexion critique sur *quoi* tester et *comment* le tester reste votre responsabilité.
*   N'hésitez pas à poser des questions si vous rencontrez des difficultés ou si certains concepts ne sont pas clairs. L'apprentissage est un processus collaboratif !

### Rendu

*   Le code source de votre application avec tous les tests implémentés (Parties 1 et 2).
*   Un document (PDF ou Markdown) contenant la discussion sur les stratégies de déploiement (Partie 3).

---