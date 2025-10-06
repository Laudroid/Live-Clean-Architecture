
# Examen Final – Clean Architecture

**Durée : 4h30**
**Consignes :**

* Vous pouvez utiliser vos notes de cours.
* Le rendu doit comporter **une partie écrite (réponses et schémas)** et **une partie pratique (code et/ou conception d’architecture)**.
* Le code peut être rédigé dans le langage/framework de votre choix (Java/Spring, C#, Node.js, etc.), mais doit respecter les principes de la Clean Architecture.

---

## Partie 1 – Questions de compréhension (1h00, 30 points)

### Q1. Rôle de l’architecte (2 pts)

Expliquez en quoi le rôle de l’architecte logiciel dépasse la simple écriture de code. Donnez **deux exemples concrets** de décisions d’architecture qui influencent la maintenabilité d’un projet à long terme.

---

### Q2. Principes SOLID (10 pts)

Pour chacun des principes SOLID, donnez :

1. Une définition claire.
2. Un exemple de violation (pseudo-code accepté).
3. Une refactorisation respectant le principe.

---

### Q3. Principes de composants (6 pts)

Associez les principes suivants à des **situations pratiques** :

* REP, CCP, CRP (cohésion)
* ADP, SDP, SAP (couplage)

Exemple attendu : « **ADP** → empêche les cycles entre modules → utile dans une architecture microservices où… »

---

### Q4. Règles de dépendance (4 pts)

Expliquez la **règle des dépendances** dans la Clean Architecture. Pourquoi dit-on que « les détails dépendent des abstractions » et pas l’inverse ?

---

### Q5. Tests et Clean Architecture (8 pts)

Proposez une stratégie de tests adaptée à un projet structuré en Clean Architecture :

* Quels types de tests pour chaque couche (entités, cas d’utilisation, adaptateurs, frameworks) ?
* Comment limiter les tests fragiles ?

---

## Partie 2 – Étude de cas et conception (1h30, 30 points)

Vous devez concevoir un **système de Master Data Management (MDM)** pour gérer les données de référence de plusieurs applications d’une entreprise.
Le système doit comporter **deux sous-systèmes complémentaires** :

* **PIM (Product Information Management)**

  * Gère la donnée de référence des produits.
  * Les produits appartiennent à une **typologie configurable** (ex. électronique, textile, mobilier).
  * L’administration doit pouvoir définir dynamiquement les **champs des formulaires** de création et de mise à jour d’un produit en fonction de la typologie.
  * Exemple : un produit de type *textile* expose des champs `taille`, `couleur`, `matière`, tandis qu’un produit de type *électronique* expose `processeur`, `RAM`, `batterie`. Le champ `prix` sera commun aux deux.

* **DAM (Digital Asset Management)**

  * Gère les médias associés aux produits (photos, vidéos, manuels PDF, etc.).
  * Le système doit être **extensible** pour accepter de nouveaux formats de média sans modification du code existant.
  * Lorsqu’un média est uploadé, il doit être **automatiquement lié au produit concerné** grâce au **code EAN** (produit) et au **code SKU** (article) présents dans le nom de fichier.
  * Exemple : fichier `EAN12345_SKU56789_front.jpg` → associé automatiquement au produit `EAN12345` et à l’article `SKU56789`.

👉 Chaque sous-système (PIM et DAM) doit être conçu en respectant les **principes de la Clean Architecture** (séparation en couches, dépendances vers l’intérieur, DIP, etc.).

---

### Tâches

#### C1. Modélisation (10 pts)

* Proposez un **diagramme d’architecture** respectant la Clean Architecture (couches, flux de données, frontières).
* Identifiez les principales **entités métier** du PIM et du DAM (au moins `Product`, `Typology`, `Media`).
* Montrez comment les données circulent entre les deux sous-systèmes (ex. upload d’un média → association avec un produit).

---

#### C2. Justification des choix (10 pts)

Expliquez :

* Comment vous appliquez **DIP** et **OCP** pour garder le système extensible (ex. nouveaux formats de médias, nouveaux types de produits).
* Comment éviter les **cycles de dépendance** (ADP) entre le PIM et le DAM.
* Quelles parties seraient testées avec des **mocks** (par ex. repository de produits dans le DAM, validation de typologies dans le PIM).

---

#### C3. Découpage en composants (10 pts)

Proposez un découpage en **modules/microservices** cohérent :

* Un module pour le **PIM** (gestion produits et typologies).
* Un module pour le **DAM** (gestion médias et association produit-média).
* Un éventuel module **MDM Core** (gestion des règles communes, ex. identifiants uniques, validation des références).

Justifiez votre découpage en vous appuyant sur les principes de **cohésion** (REP, CCP, CRP) et de **couplage** (ADP, SDP, SAP).

---

👉 Résultat attendu :
À la fin, vous devez être capable de montrer :

* un **schéma d’architecture** clair,
* une **explication argumentée** de vos choix,
* un **découpage en composants/microservices** respectant la Clean Architecture.

---


## Partie 3 – Réalisation pratique (2h00, 40 points)

Vous allez réaliser un **prototype applicatif** (sans contrainte de langage/framework, au choix Java, C#, Python, Node.js, etc.) correspondant à l’étude de cas définie en **Partie 2** (MDM avec PIM et DAM).

⚠️ **Consignes générales** :

* Respecter les **règles de la Clean Architecture** (séparation claire Entités / Use Cases / Interfaces / Frameworks & Drivers).
* Penser à l’**indépendance des frameworks** (ex. pas de dépendance directe entre vos cas d’usage et votre ORM / API REST / UI).
* Appliquer **SOLID** et **principes de composants (ADP, CCP, CRP, DIP, SAP, SDP)**.
* Prévoir une **gestion de la sécurité minimale** (authentification ou contrôle des accès).
* Prévoir des **tests unitaires et/ou d’intégration** pour valider vos cas d’usage.
* (Bonus, +2 points) Décrire une stratégie simple de **déploiement** (conteneurs, CI/CD, microservices).

---

### **Travail demandé**

1. **Modélisation des entités et use cases (10 points)**

   * Définissez les **entités principales du PIM** (ex. `Produit`, `TypologieProduit`, `AttributDynamique`) et du DAM (ex. `Media`, `FormatMedia`, `LienProduitMedia`).
   * Rédigez au moins **2 cas d’utilisation** pour le PIM (ex. *Créer un produit selon sa typologie*, *Mettre à jour un produit avec des attributs dynamiques*).
   * Rédigez au moins **2 cas d’utilisation** pour le DAM (ex. *Uploader une série de médias*, *Associer automatiquement un média à un produit via code EAN/SKU*).

   ✅ **Livrable attendu** : diagramme ou pseudo-code + brève description des cas d’usage.

---

2. **Implémentation du système**

Vous devez démontrer votre capacité à **implémenter une partie du système MDM** en respectant les principes de la Clean Architecture.

### Travail attendu :

1. **Implémentation PIM (8 points)**

   * Définir les **interfaces et classes concrètes** nécessaires au cas d’utilisation *Créer un produit selon sa typologie*.
   * Montrer comment vous gérez les **formulaires dynamiques** (par ex. : un `FormDefinition` décrit les attributs configurables d’un produit, et le use case génère la structure attendue).
   * Expliquer où interviennent **DIP** (interfaces pour référentiels, injection dans les interactors) et **OCP** (ajout de nouvelles typologies sans modifier le code existant).

   ✅ **Livrable attendu** :

   * Schéma de packages ou diagramme de classe simple.
   * Code (interfaces + classes + exemple d’instanciation).

---

2. **Implémentation DAM (8 points)**

   * Définir les **interfaces et classes concrètes** pour le cas d’utilisation *Uploader une série de médias et l’associer automatiquement à un produit*.
   * Décrire le **mécanisme de liaison automatique** produit–média (utilisation du code EAN et SKU parsés depuis le nom du fichier).
   * Expliquer l’application du principe **ADP** (pas de dépendance cyclique entre PIM et DAM : la liaison passe via un service MDM Core ou une interface abstraite).

   ✅ **Livrable attendu** :

   * Schéma de packages ou diagramme de classe simple.
   * Code (interfaces + classes + exemple d’instanciation).

---

3. **Implémentation MDM Core (6 points)**

   * Montrer comment vous centralisez les **règles de cohérence** entre PIM et DAM dans un **MDM Core**.
   * Définir les **boundaries/interfaces** qui permettent au Core d’orchestrer les appels (ex. `IMdmOrchestrator`, `IEventPublisher`).
   * Expliquer comment vous appliquez **DIP** (Core dépend d’abstractions, pas des implémentations concrètes de PIM ou DAM).
   * Expliquer comment vous appliquez **OCP** (extension possible à de nouveaux domaines de données de référence).

   ✅ **Livrable attendu** :

   * Schéma global (MDM Core ↔ PIM ↔ DAM).
   * Exemple d’interface partagée (`IProductService`, `IMediaLinkService`).
   * Code (interfaces + classes + exemple d’orchestrateur MDM qui reçoit un événement produit/dam et gère la synchronisation.)

---

3. **Sécurité, tests et déploiement (8 points)**

   * **Sécurité (3 points)** : proposez une approche simple (authentification JWT, RBAC par rôle admin/utilisateur).
   * **Tests (3 points)** : rédigez un test unitaire pour un cas d’utilisation du PIM ou du DAM (pseudo-code accepté).
   * **Déploiement (2 points, bonus)** : décrivez une stratégie simple (microservices Dockerisés, orchestrés par Kubernetes ou docker-compose).

   ✅ **Livrable attendu** :

   * Brève description de l’approche sécurité.
   * Exemple de test unitaire.
   * Description textuelle d’un pipeline de déploiement.

---



