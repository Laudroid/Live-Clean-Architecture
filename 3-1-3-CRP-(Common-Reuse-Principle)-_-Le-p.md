# CRP (Common Reuse Principle) : Le principe de réutilisation commune

Le CRP (Common Reuse Principle) est un principe clé de la conception de composants logiciels, introduit par Robert C. Martin. Il complète l’idée de modularité en mettant l’accent sur un aspect critique pour la réutilisation : **les clients d’un composant doivent être capables de réutiliser une partie sans avoir à prendre le tout**.

---

## 1. Définition du CRP

> Un composant ne doit être réutilisé que dans son intégralité ou pas du tout. On évite que l’utilisation d’une partie entraîne la nécessité d’importer des parties inutilisées d’un composant.

Autrement dit, **les classes regroupées dans un composant doivent être liées par un même objectif de réutilisation**. Si certains éléments ne sont jamais réutilisés ensemble, ils doivent résider dans des composants distincts.

---

## 2. Pourquoi respecter le CRP ?

Sans respect du CRP, on se retrouve avec des composants maladroits :

- Les utilisateurs doivent importer des éléments inutiles, générant un gaspillage.
- Les dépendances se complexifient, rendant le système plus lourd.
- Les mises à jour ou corrections d’éléments non utilisés peuvent impacter inutilement les clients.
  
Le CRP permet donc d’**optimiser la granularité des composants** en fonction des besoins réels de réutilisation.

---

## 3. Exemple concret

### Mauvaise application du CRP

Un composant `Utilitaires` regroupe à la fois :

- Des fonctions de calcul mathématique.
- Des outils de manipulation de fichiers.
- Des helpers pour le parsing XML.

Un projet qui a seulement besoin de fonctions mathématiques doit importer tout ce composant, incluant les dépendances de fichiers et XML, inutile et potentiellement problématique.

---

### Respect du CRP

- Composant `MathUtils` : fonctions mathématiques pures.
- Composant `FileUtils` : gestion de fichiers.
- Composant `XMLParser` : parsing XML.

Chaque client importe uniquement ce dont il a besoin, ce qui améliore la clarté, réduit le couplage et simplifie la maintenance.

---

## 4. Diagramme Mermaid illustrant le CRP

```mermaid
graph TD
    subgraph MathUtils
        M1[MathOperations]
        M2[AlgebraHelpers]
    end
    subgraph FileUtils
        F1[FileReader]
        F2[FileWriter]
    end
    subgraph XMLParser
        X1[XMLParserCore]
        X2[XMLValidator]
    end
    Client --> MathUtils
    Client --x--> FileUtils
    Client --x--> XMLParser
```

Ici, le client dépend uniquement de `MathUtils`, illustrant la séparation claire des composants selon leurs domaines de réutilisation.

---

## 5. Conséquences d’une violation du CRP

- Impossibilité d’utiliser une portion isolée d’un composant sans embarquer tout le reste.
- Augmentation des dépendances inutiles, impactant la taille, les performances et la complexité.
- Diminution de la flexibilité liée au partage et à la maintenance du code.

---

## 6. Recommandations pour appliquer le CRP

- Analyser les usages et besoins réels des composants dans les différents projets.
- Décomposer les composants pour isoler les fonctionnalités réutilisées ensemble.
- Éviter d’agréger dans un même composant des fonctionnalités disparates.
- Mettre en place une gestion rigoureuse des dépendances.

---

## Sources

- Robert C. Martin, *Agile Software Development, Principles, Patterns, and Practices* (2002)  
- [Clean Architecture - Uncle Bob: Component Principles](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-component-principles.html)  
- [Refactoring.Guru - Component Principles](https://refactoring.guru/design-principles/component-principles)  
- [NDepend - Component Principles](https://www.ndepend.com/docs/component-principles)  

---

Le CRP aide à rendre les composants logiciels plus cohésifs du point de vue de la réutilisation, en assurant que ceux-ci regroupent uniquement ce qui est réellement utilisé ensemble. Ce principe est fondamental pour la conception de systèmes modulaires, performants et faciles à maintenir.