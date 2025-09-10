# SDP (Stable Dependencies Principle) : Le principe des dépendances stables

Le SDP (Stable Dependencies Principle) fait partie des principes fondamentaux pour organiser le couplage des composants logiciels dans une architecture modulaire. Il guide la gestion des dépendances entre composants en privilégiant la stabilité des modules de base.

---

## 1. Définition du SDP

> Un composant doit dépendre uniquement de composants plus stables que lui.

Autrement dit, les dépendances doivent pointer vers des composants **moins susceptibles de changer**. Ainsi, un composant instable peut évoluer librement, tandis que ses dépendances stables protègent la cohérence globale du système.

---

## 2. Mesure de la stabilité

Pour appliquer le SDP, Robert C. Martin propose une métrique appelée **stabilité**, calculée ainsi :

\[
I = \frac{Ce}{Ce + Ca}
\]

- \(Ce\): Couverture sortante — le nombre de composants dont dépend le composant (sorties)
- \(Ca\): Couverture entrante — le nombre de composants qui dépendent du composant (entrées)

Un composant est **stable** quand il est beaucoup dépendu (grand \(Ca\)) et dépend peu (petit \(Ce\)) — ce qui donne un indice \(I\) proche de 0. À l’inverse, un composant avec beaucoup de dépendances mais peu utilisé est instable (\(I\) proche de 1).

---

## 3. Pourquoi respecter le SDP ?

- Garantir que les changements dans les composants instables n’entrainent pas de modifications massives dans les dépendances.
- Assurer une architecture robuste où les composants de base sont fiables et peu modifiés.
- Réduire le risque de cascade d’effets lors de l’évolution du système.

---

## 4. Exemple de respect / non-respect du SDP

### Non-respect

- Un composant très stable (fortement dépendu par d’autres) dépend d’un composant instable (qui change fréquemment).  
- Cela expose le système à des risques de régressions multiples.

### Respect

- Les composants stables sont positionnés en bas de la hiérarchie des dépendances.
- Les composants instables dépendent des stables, évitant ainsi que des éléments peu fiables propagent leurs changements.

---

## 5. Illustration diagramme Mermaid

```mermaid
graph TD
    StableBase[Composant Stable<br>(I proche de 0)]
    Intermediate[Composant Intermédiaire]
    UnstableTop[Composant Instable<br>(I proche de 1)]

    UnstableTop --> Intermediate
    Intermediate --> StableBase
```

Les dépendances vont des composants instables vers des couches plus stables, conformément au SDP.

---

## 6. Conseils pour appliquer le SDP

- Identifier la stabilité relative des composants via leur utilisation et leurs dépendances.
- Favoriser les abstractions dans les composants stables pour les rendre flexibles.
- Éviter les dépendances inverses où un composant stable dépend d’un composant instable.
- Utiliser des outils d’analyse de dépendances pour visualiser et corriger les violations du SDP.

---

## 7. Sources

- Robert C. Martin, *Agile Software Development, Principles, Patterns, and Practices* (2002)  
- [Uncle Bob - The Dependency Inversion Principle](https://blog.cleancoder.com/uncle-bob/2014/05/15/DependencyRejection.html)  
- [Martin Fowler - DependencyOverhead](https://martinfowler.com/articles/components.html)  
- [NDepend - Stable Dependencies Principle](https://www.ndepend.com/docs/component-stability-metrics)  

---

Le SDP aide à construire des architectures logicielles où les dépendances sont hiérarchisées selon la stabilité, minimisant les impacts des changements et assurant une meilleure résilience des composants, pierre angulaire d’une conception modulaire saine.