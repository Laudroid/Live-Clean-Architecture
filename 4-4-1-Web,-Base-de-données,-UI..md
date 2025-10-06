# Frameworks et Pilotes : Web, Base de Données et UI dans la Clean Architecture

Dans la Clean Architecture, la couche la plus externe regroupe les **frameworks et pilotes** (Frameworks & Drivers). Cette couche contient les éléments techniques qui interagissent avec le monde extérieur : applications web, bases de données, interfaces utilisateur, et autres systèmes externes. Comprendre leur rôle et leur intégration garantit que la logique métier reste indépendante des technologies et facilement testable.

---

## 1. Fonction de la couche Frameworks & Drivers

Cette couche **encapsule la complexité technique** liée aux plateformes et outils sous-jacents, tout en respectant le principe d’**inversion des dépendances** :

- **Accepter les appels entrants** (HTTP, UI événements).  
- **Exécuter les appels sortants vers stockages** (bases de données, systèmes de fichiers) ou services externes.  
- Adapter les interfaces techniques à la couche métier via des adaptateurs.

Elle ne contient pas de règles métier, mais soutient l’exécution de la logique en exposant ses points d’entrée et de sortie.

---

## 2. Exemple des sous-couches clés

### 2.1 Framework Web

- Fournit la gestion des requêtes HTTP (ex. ASP.NET Core, Spring Web, Express.js).  
- Gère le routage, la sérialisation, la gestion des sessions.  
- Implémente généralement les **contrôleurs** qui reçoivent les requêtes et invoquent les use cases.

**Exemple simple en ASP.NET Core :**

```csharp
[ApiController]
[Route("api/[controller]")]
public class CommandesController : ControllerBase
{
    private readonly IValiderCommandeUseCase useCase;

    public CommandesController(IValiderCommandeUseCase useCase) 
    {
        this.useCase = useCase;
    }

    [HttpPost("{id}/valider")]
    public IActionResult Valider(int id) 
    {
        try
        {
            useCase.Exécuter(id);
            return Ok();
        }
        catch(Exception ex)
        {
            return BadRequest(ex.Message);
        }
    }
}
```

---

### 2.2 Base de données

- Contient les implémentations de **repositories** ou **gateways**.  
- Utilise les ORM (Entity Framework, Hibernate), clients SQL ou autres technologies.  
- Traduit entre modèles métiers et tables/collections en base.

**Exemple d’implémentation repository :**

```csharp
public class CommandeRepository : ICommandeRepository
{
    private readonly DbContext context;

    public CommandeRepository(DbContext context)
    {
        this.context = context;
    }

    public Commande GetById(int id) => context.Commandes.Find(id);

    public void Save(Commande commande)
    {
        context.Commandes.Update(commande);
        context.SaveChanges();
    }
}
```

---

### 2.3 Interface Utilisateur (UI)

- Peut être une application web (React, Angular), une application mobile (Flutter, Xamarin) ou un client desktop.  
- Communique généralement avec l’API via HTTP ou d’autres protocoles.  
- Consomme les données formatées par les présentateurs.

**Exemple simple en React affichant un état de commande :**

```jsx
function StatutCommande({ idCommande }) {
  const [statut, setStatut] = React.useState(null);

  React.useEffect(() => {
    fetch(`/api/commandes/${idCommande}/statut`)
      .then(res => res.json())
      .then(data => setStatut(data.statut));
  }, [idCommande]);

  return <div>Statut de la commande : {statut}</div>;
}
```

---

## 3. Diagramme Mermaid résumé

```mermaid
graph TD
    UI[UI - Front-end (React, Angular)]
    Web[Framework Web (ASP.NET Core, Spring)]
    UseCase[Cas d'Utilisation]
    Repo[Repository/Gateway]
    DB[Base de Données]

    UI -- HTTP Requête --> Web
    Web -- Appel Méthode --> UseCase
    UseCase -- Interaction --> Repo
    Repo -- Requête --> DB
    DB -- Résultat --> Repo
    Repo -- Entité --> UseCase
    UseCase -- Résultat --> Web
    Web -- Réponse HTTP --> UI
```

---

## 4. Conseils pour intégrer Frameworks et Pilotes

- Ne pas laisser de règles métier dans cette couche.  
- Utiliser des interfaces abstraites dans la couche de domaine ou use cases et injecter les implémentations concrètes via la couche frameworks.  
- Garder la dépendance dirigée vers l’intérieur : frameworks et pilotes dépendent du domaine, jamais l’inverse.  
- Tester séparément la logique métier indépendamment des frameworks.

---

## 5. Sources

- Robert C. Martin, *Clean Architecture*, 2017  
- Microsoft Docs, [Clean architecture in ASP.NET Core](https://docs.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures#clean-architecture)  
- Uncle Bob, [The Clean Architecture Explained](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)  
- Simon Brown, *Software Architecture for Developers*, 2014

---

La couche Frameworks & Pilotes constitue le point de contact avec la technologie et le monde extérieur. En conservant ses responsabilités limitées à la gestion technique et à l’adaptation aux interfaces, elle permet à la Clean Architecture d’offrir une solution robuste, évolutive, et maintenable.