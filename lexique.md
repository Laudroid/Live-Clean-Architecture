**Abstractions** : Utilisées par le OCP pour concevoir des entités extensibles sans modification, et par le DIP pour inverser les dépendances et dépendre d'elles plutôt que des concrétions.

**Adaptateurs d'Interface (Interface Adapters)** : Une des couches de la Clean Architecture, incluant Présentateurs, Gateways et Contrôleurs, responsable de la conversion de données entre les couches.

**ADP (Acyclic Dependencies Principle)** : Un des principes de couplage des composants, concernant les dépendances acycliques.

**Agences Externes** : Un des éléments dont la Clean Architecture garantit l'indépendance.

**Architecte logiciel** : Rôle et responsabilités dans un projet de développement complexe, notamment en matière d'architecture logicielle.

**Architecture logicielle** : Concerne la séparation efficace des composants d'une application et la différenciation des éléments essentiels des accessoires.

**Assemblies** : Un type de regroupement pour organiser les composants d'une application.

**Base de Données** : Un des éléments dont la Clean Architecture garantit l'indépendance; elle est considérée comme un détail d'implémentation externe pour la couche Frameworks & Pilotes.

**Bonnes Pratiques** : Conseils pour une architecture durable lors de l'implémentation de la Clean Architecture.

**Boundary** : Concept décrivant comment les données traversent les couches de l'architecture, via les interfaces et les DTOs.

**Cas d'Utilisation (Use Cases / Interactors)** : Une des couches de la Clean Architecture, contenant les règles métier spécifiques à l'application et assurant l'orchestration du flux de données.

**CCP (Common Closure Principle)** : Un des principes de cohésion des composants.

**Clean Architecture** : Approche résolvant les problèmes de couplage fort et de dépendances non contrôlées, visant à améliorer la maintenabilité, la testabilité et la flexibilité, basée sur l'indépendance et les règles de dépendance entre ses couches.

**Cohésion** : Degré de regroupement des responsabilités dans un module; impactée par le Principe de Responsabilité Unique (SRP).

**Composants** : Éléments d'une application dont la séparation efficace favorise la modularité et la réutilisation, et dont la cohésion et le couplage sont régis par des principes spécifiques.

**Concrétions** : Implémentations spécifiques dont le Principe d'Inversion de Dépendance (DIP) recommande de ne pas dépendre, préférant les abstractions.

**Considérations de Déploiement** : Réflexions et stratégies pour le déploiement d'applications basées sur la Clean Architecture, incluant la conteneurisation et les microservices.

**Contrats** : Ensemble d'attentes et de garanties que les sous-types doivent respecter, concept central du Principe de Substitution de Liskov (LSP).

**Conteneurisation** : Stratégie de déploiement pour les applications basées sur la Clean Architecture.

**Contrôleurs** : Faisant partie des Adaptateurs d'Interface, ils participent à la conversion de données entre les couches.

**Conversion de données entre les couches** : Rôle principal des Adaptateurs d'Interface dans la Clean Architecture.

**Couches de la Clean Architecture** : La structure fondamentale de la Clean Architecture, composée des Entités, Cas d'Utilisation, Adaptateurs d'Interface, et Frameworks & Pilotes.

**Couplage** : Degré d'interdépendance entre les composants; un 'couplage fort' est un problème courant résolu par la Clean Architecture et impacté par le SRP.

**CRP (Common Reuse Principle)** : Un des principes de cohésion des composants.

**Dépendances non contrôlées** : Problème courant dans les architectures logicielles traditionnelles que la Clean Architecture vise à résoudre.

**Déploiement** : Processus de mise à disposition d'une application, avec des stratégies spécifiques pour les architectures basées sur la Clean Architecture.

**DIP (Dependency Inversion Principle)** : Un des principes SOLID et de couplage des composants, consistant à inverser les dépendances pour dépendre des abstractions plutôt que des concrétions, et introduisant le concept d'injection de dépendances.

**DTOs (Data Transfer Objects)** : Objets utilisés pour le passage des données à travers les couches de l'architecture, notamment au niveau du 'Boundary'.

**Éléments essentiels des accessoires** : Distinction cruciale à faire dans une architecture logicielle pour favoriser la robustesse et la flexibilité.

**Entités (Entities)** : La couche la plus interne de la Clean Architecture, contenant les règles métier d'entreprise et les structures de données essentielles.

**Flexibilité** : Un des bénéfices attendus de l'adoption de la Clean Architecture.

**Frameworks** : Un des éléments dont la Clean Architecture garantit l'indépendance, les traitant comme des détails d'implémentation externes.

**Frameworks & Pilotes (Frameworks & Drivers)** : La couche la plus externe de la Clean Architecture, gérant les détails d'implémentation externes comme le Web, la Base de Données et l'UI.

**Gateways** : Faisant partie des Adaptateurs d'Interface, elles facilitent la conversion de données entre les couches.

**Gérer les dépendances entre les composants** : Une stratégie clé pour la séparation efficace des composants d'une application.

**Indépendance des Agences Externes** : Un des principes fondamentaux de la Clean Architecture, assurant que le cœur de l l'application ne dépend pas d'éléments tiers.

**Indépendance de la Base de Données** : Un des principes fondamentaux de la Clean Architecture, assurant que le cœur de l'application n'est pas lié à un système de base de données spécifique.

**Indépendance de l'UI** : Un des principes fondamentaux de la Clean Architecture, assurant que le cœur de l'application est indépendant de son interface utilisateur.

**Indépendance des Frameworks** : Un des principes fondamentaux de la Clean Architecture, assurant que le cœur de l'application ne dépend pas d'un framework spécifique.

**Injection de dépendances** : Concept introduit par le Principe d'Inversion de Dépendance (DIP), permettant d'injecter les dépendances plutôt que de les créer directement.

**Interfaces** : Éléments jouant un rôle crucial dans le concept de 'Boundary' pour le passage des données entre les couches; leur ségrégation est un principe clé (ISP).

**Interfaces "grosses" (fat interfaces)** : Interfaces contenant de nombreuses méthodes inutiles pour certains clients, à éviter selon le Principe de Ségrégation d'Interface (ISP).

**ISP (Interface Segregation Principle)** : Un des principes SOLID et de couplage des composants, visant à éviter les interfaces 'grosses' et à assurer la spécificité des interfaces pour les clients.

**LSP (Liskov Substitution Principle)** : Un des principes SOLID et de couplage des composants, concernant la compréhension des contrats et les attentes des sous-types.

**Maintenabilité du code** : Objectif clé des principes fondamentaux de conception logicielle et un bénéfice attendu de la Clean Architecture.

**Microservices** : Stratégie de déploiement pour les applications basées sur la Clean Architecture.

**Modularité** : Objectif favorisé par la séparation efficace des composants d'une application.

**Modules** : Un type de regroupement pour organiser les composants d'une application.

**OCP (Open/Closed Principle)** : Un des principes SOLID et de couplage des composants, expliquant comment concevoir des entités extensibles sans modification grâce aux abstractions.

**Orchestration du flux de données** : Fonction principale des Cas d'Utilisation dans la Clean Architecture.

**Packages** : Un type de regroupement pour organiser les composants d'une application.

**Pièges Courants** : Erreurs à éviter lors de l'implémentation de la Clean Architecture.

**Présentateurs** : Faisant partie des Adaptateurs d'Interface, ils participent à la conversion de données entre les couches.

**Principes de Cohésion des Composants** : Ensemble de principes (REP, CCP, CRP) guidant la bonne structuration des composants.

**Principes de Couplage des Composants** : Ensemble de principes (ADP, SDP, SAP) guidant la gestion des dépendances entre les composants.

**Principes fondamentaux de conception logicielle** : Ensemble de règles de base pour améliorer la qualité et la maintenabilité du code.

**Principes SOLID** : Ensemble de cinq principes (SRP, OCP, LSP, ISP, DIP) fondamentaux pour la conception logicielle, visant à améliorer la qualité et la maintenabilité du code.

**Principe de Responsabilité Unique (SRP)** : Un des principes SOLID, définissant l'impact sur la cohésion et le couplage du code.

**Qualité** : Un des objectifs des principes fondamentaux de conception logicielle et un bénéfice de la Clean Architecture, notamment en termes de qualité du code.

**Règles de dépendance** : Principes fondamentaux de la Clean Architecture régissant la manière dont les couches peuvent dépendre les unes des autres.

**Règles métier d'entreprise (Enterprise Business Rules)** : Règles fondamentales et essentielles au fonctionnement de l'entreprise, contenues dans la couche Entités.

**Règles métier spécifiques à l'application (Application Business Rules)** : Règles métier propres à une application donnée, contenues dans la couche Cas d'Utilisation.

**REP (Release/Reuse Equivalence Principle)** : Un des principes de cohésion des composants.

**Réutilisation** : Objectif favorisé par la séparation efficace des composants d'une application.

**SAP (Stable Abstractions Principle)** : Un des principes de couplage des composants, concernant les abstractions stables.

**SDP (Stable Dependencies Principle)** : Un des principes de couplage des composants, concernant les dépendances stables.

**Séparation des Composants** : La capacité à diviser une application en parties distinctes pour favoriser la modularité et la réutilisation, régie par des stratégies spécifiques.

**Spécificité des interfaces pour les clients** : Objectif du Principe de Ségrégation d'Interface (ISP) pour éviter les interfaces 'grosses'.

**Structures de données essentielles** : Éléments fondamentaux de l'information de l'entreprise, contenus dans la couche Entités.

**Sous-types** : Classes qui héritent ou implémentent des types parents, dont les contrats et attentes sont à comprendre dans le contexte du Principe de Substitution de Liskov (LSP).

**Tendances** : Évolutions récentes dans le domaine de la Clean Architecture.

**Testabilité** : Un des bénéfices attendus de l'adoption de la Clean Architecture, permettant de tester chaque couche de manière isolée.

**Tests** : Processus d'évaluation comprenant des stratégies unitaires, d'intégration et de bout en bout, pour vérifier le bon fonctionnement de l'application dans la Clean Architecture.

**Tests de bout en bout** : Stratégie de test pour vérifier le système dans son intégralité, dans le cadre de la Clean Architecture.

**Tests d'intégration** : Stratégie de test pour vérifier l'interaction entre les différentes couches ou composants, dans le cadre de la Clean Architecture.

**Tests unitaires** : Stratégie de test pour vérifier chaque petite unité de code de manière isolée, dans le cadre de la Clean Architecture.

**UI (User Interface)** : Un des éléments dont la Clean Architecture garantit l'indépendance; elle est considérée comme un détail d'implémentation externe pour la couche Frameworks & Pilotes.

**Web** : Un des détails d'implémentation externes géré par la couche Frameworks & Pilotes de la Clean Architecture.

