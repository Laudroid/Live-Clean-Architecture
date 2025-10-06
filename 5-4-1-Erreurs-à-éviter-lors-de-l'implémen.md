# Erreurs à éviter lors de l'implémentation de la Clean Architecture

La mise en œuvre de la Clean Architecture vise à structurer le code en couches bien séparées, favorisant maintenabilité et testabilité. Pourtant, certains pièges fréquents compromettent ces bénéfices. Cet article identifie ces erreurs courantes et propose des bonnes pratiques pour les éviter, sur la base d’expériences récentes et de recommandations d’experts.

---

## 1. Couche Domaine non isolée, interactions directes avec l’infrastructure

### Problème

La couche Domaine doit être complètement indépendante des détails techniques (base de données, frameworks). Certains confondent cette règle et introduisent des dépendances directes, par exemple en appelant directement l’ORM ou en manipulant des DTO spécifiques dans le domaine.

### Conséquences

- Perte de portabilité et de testabilité.  
- Difficulté à maintenir la logique métier isolée.  

### Solution

Définir dans le Domaine uniquement des entités et interfaces abstraites. Les implémentations techniques doivent rester dans l'Infrastructure.

---

## 2. Absence ou mauvaise utilisation des interfaces (ports)

### Problème

Ignorer la définition d’interfaces pour les dépendances vers l’extérieur implique un couplage fort et rend les tests isolés difficiles.

### Exemple mauvais

```csharp
// Couche Application dépend directement d’une classe concrète
public class CommandeUseCase {
    private SqlCommandeRepository repo = new SqlCommandeRepository();
}
```

### Bon usage

```csharp
public class CommandeUseCase {
    private ICommandeRepository repo;
    public CommandeUseCase(ICommandeRepository repo) {
        this.repo = repo;
    }
}
```

Ce pattern facilite le mock des dépendances en test unitaire.

---

## 3. Surcharge des couches : mettre trop de logique côté Interface Utilisateur

Mettre la validation métier, règles complexes ou logique métier dans des contrôleurs ou composants UI conduit à dupliquation et difficulté à évoluer.

---

## 4. Trop de dépendances dirigées vers l’extérieur

Ne pas respecter la règle que les dépendances doivent pointer vers l’intérieur (couche Domaine) mais à l'inverse, cela empêche la stabilité et la modularité.

---

## 5. Mauvaise gestion des exceptions et des erreurs

Éviter d'exposer les exceptions issues des couches infrastructurelles directement à l’utilisateur ou couche supérieure. Utiliser des adaptateurs/gestionnaires pour transformer les erreurs en messages contrôlés.

---

## 6. Exemple Mermaid illustrant les mauvais couplages fréquents

```mermaid
graph TD
    UI[Interface Utilisateur]
    App[Application - Use Cases]
    Domain[Couche Domaine]
    Infra[Infrastructure]

    UI --> App
    App --> Infra
    Infra --> Domain  %% Mauvais sens: infra ne doit pas dépendre du domaine
    Domain --> Infra  %% Mauvais sens: domaine dépend d'infrastructure (à éviter)
```

Les dépendances correctes doivent toujours orienter vers l’intérieur (Domain).

---

## 7. Bonnes pratiques synthétiques

- Respecter l’inversion des dépendances: dépendances pointent toujours vers les couches supérieures (Domaine).  
- Utiliser des **interfaces** comme abstraction des dépendances externes.  
- Garder le **Domaine pur**, sans package technique, sans framework.  
- Implémenter les tests unitaires pour valider isolément chaque couche.  
- Appliquer une gestion uniforme et centralisée des erreurs.

---

## 8. Sources et références

- Robert C. Martin, *Clean Architecture*, 2017  
- Microsoft Docs, [Architectures de référence .NET](https://docs.microsoft.com/fr-fr/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures#clean-architecture)  
- Martin Fowler, [The Dependency Inversion Principle](https://martinfowler.com/bliki/DependencyInversionPrinciple.html)  
- Clean Architecture en pratique, https://blog.ploeh.dk/2018/02/15/clean-architecture-in-practice/  

---

Respecter ces principes évite des écueils classiques qui affaiblissent la robustesse et la qualité d’une application. Le respect rigoureux des couches et des dépendances garantit que la Clean Architecture délivre toute sa valeur.