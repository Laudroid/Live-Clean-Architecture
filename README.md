# Cours Clean Architecture

## Objectifs de la formation

- Comprendre le rôle et les responsabilités de l'architecte logiciel dans un projet de développement complexe.
- Appliquer les principes fondamentaux de conception logicielle pour améliorer la qualité et la maintenabilité du code.
- Maîtriser les concepts SOLID et leur application concrète.
- S'être familiarisé avec les principes ADP, CCP, CRP, DIP, ISP, LSP, OCP, REP, SAP et SDP.
- Savoir séparer efficacement les composants d'une application pour favoriser la modularité et la réutilisation.
- Être capable de différencier les éléments essentiels des accessoires dans une architecture logicielle.

  
## Séance 1 : Introduction à l'Architecture Logicielle et aux Fondamentaux de la Clean Architecture

### Objectifs pédagogiques

- Comprendre le rôle de l'architecte logiciel.
- Saisir les principes fondamentaux de conception logicielle et les problèmes résolus par la Clean Architecture.
- Maîtriser les concepts de base et les couches de la Clean Architecture.

### Contenus

#### Introduction à l'Architecture Logicielle

- Qu'est-ce que l'architecture logicielle ?
- Rôle et responsabilités de l'architecte logiciel dans un projet de développement complexe.
#### Les Problèmes Résolus par la Clean Architecture

- Identification des problèmes courants dans les architectures logicielles traditionnelles (couplage fort, dépendances non contrôlées).
- Pourquoi la Clean Architecture ? Les bénéfices attendus (maintenabilité, testabilité, flexibilité).

#### Principes Fondamentaux de la Clean Architecture

- Indépendance des Frameworks, de l'UI, de la Base de Données et des Agences Externes.
- Les règles de dépendance.
- Les couches de la Clean Architecture : Entités, Cas d'Utilisation, Adaptateurs d'Interface, Frameworks & Pilotes.

## Séance 2 : Maîtrise des Principes SOLID

### Objectifs pédagogiques

- Maîtriser les concepts SOLID.
- Appliquer concrètement les principes SOLID pour améliorer la qualité et la maintenabilité du code.

### Contenus

#### Le Principe de Responsabilité Unique (SRP)

- Définition et exemples concrets.
- Impact sur la cohésion et le couplage.

#### Le Principe Ouvert/Fermé (OCP)

- Comment concevoir des entités extensibles sans modification ?
- Utilisation des abstractions.

#### Le Principe de Substitution de Liskov (LSP)

- Comprendre les contrats et les attentes des sous-types.
- Conséquences d'une violation de LSP.

#### Le Principe de Ségrégation d'Interface (ISP)

- Éviter les interfaces "grosses" (fat interfaces).
- Spécificité des interfaces pour les clients.

#### Le Principe d'Inversion de Dépendance (DIP)

- Inverser les dépendances pour dépendre des abstractions plutôt que des concrétions.
- Introduction au concept d'injection de dépendances.

## Séance 3 : Principes de Composants et Gestion des Dépendances

### Objectifs pédagogiques

- Se familiariser avec les principes ADP, CCP, CRP, DIP, ISP, LSP, OCP, REP, SAP et SDP.
- Savoir séparer efficacement les composants d'une application pour favoriser la modularité et la réutilisation.

### Contenus

#### Principes de Cohésion des Composants

- REP (Release/Reuse Equivalence Principle) : Le principe d'équivalence de la réutilisation et de la publication.
- CCP (Common Closure Principle) : Le principe de clôture commune.
- CRP (Common Reuse Principle) : Le principe de réutilisation commune.

#### Principes de Couplage des Composants

- ADP (Acyclic Dependencies Principle) : Le principe des dépendances acycliques.
- SDP (Stable Dependencies Principle) : Le principe des dépendances stables.
- SAP (Stable Abstractions Principle) : Le principe des abstractions stables.

#### Stratégies de Séparation des Composants

- Comment organiser les composants (packages, modules, assemblies).
- Gérer les dépendances entre les composants.

## Séance 4 : Blocs de Construction de la Clean Architecture et Application Pratique

### Objectifs pédagogiques

- Différencier les éléments essentiels des accessoires dans une architecture logicielle.
- Appliquer les concepts de la Clean Architecture dans un contexte pratique.

### Contenus

#### Les Entités (Entities)

- Règles métier d'entreprise (Enterprise Business Rules).
- Structures de données essentielles.

#### Les Cas d'Utilisation (Use Cases / Interactors)

- Règles métier spécifiques à l'application (Application Business Rules).
- Orchestration du flux de données.

#### Les Adaptateurs d'Interface (Interface Adapters)

- Présentateurs, Gateways, Contrôleurs.
- Conversion de données entre les couches.

#### Les Frameworks et Pilotes (Frameworks & Drivers)

- Web, Base de données, UI.
- Détails d'implémentation externes.

#### Le Concept de "Boundary"

- Comment les données traversent les couches de l'architecture.
- Le rôle des interfaces et des DTOs.

## Séance 5 : Sujets Avancés, Tests et Considérations de Déploiement

### Objectifs pédagogiques

- Consolider la compréhension de la Clean Architecture.
- Aborder les stratégies de test et de déploiement spécifiques à la Clean Architecture.
- Prendre connaissance des dernières mises à jour et meilleures pratiques.

### Contenus

#### Tests dans la Clean Architecture

- Stratégies de tests unitaires, d'intégration et de bout en bout.
- Comment tester chaque couche de manière isolée.

#### Considérations de Déploiement

- Stratégies de déploiement pour les applications basées sur la Clean Architecture.
- Conteneurisation et microservices.

#### Application de la Clean Architecture avec Différentes Technologies

- Exemples d'intégration avec des frameworks modernes (ASP.NET Core, Node.js, Spring Boot, Next.js 14, etc.).
- Adaptation des principes aux spécificités technologiques.

#### Pièges Courants et Bonnes Pratiques

- Erreurs à éviter lors de l'implémentation.
- Conseils pour une architecture durable.

#### Dernières Mises à Jour et Tendances

- Revue des évolutions récentes dans le domaine de la Clean Architecture.
- Discussion ouverte et Q&A.
