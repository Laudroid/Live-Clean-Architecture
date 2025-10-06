# Application de la Clean Architecture avec des frameworks modernes : ASP.NET Core, Node.js, Spring Boot, Next.js 14

La Clean Architecture propose un cadre conceptuel structurant les applications en couches indépendantes, facilitant leur maintenabilité et testabilité. Son intégration dans des environnements technologiques variés (ASP.NET Core, Node.js, Spring Boot, Next.js 14) est possible et bénéfique. Cet article illustre comment appliquer ce modèle dans ces frameworks, avec exemples et bonnes pratiques.

---

## 1. ASP.NET Core : architecture modulaire et injection de dépendances

### Intégration

- La séparation en couches (Domain, Application, Infrastructure, API/Web) se prête naturellement au modèle MVC ou API ASP.NET Core.  
- Contrôleurs exposent les **use cases** via interfaces injectées par DI.  
- Entity Framework Core sert souvent d’implémentation Infrastructure des repositories.

### Exemple résumé

```csharp
// Interface use case
public interface IGetUserUseCase
{
    UserDto Execute(Guid userId);
}

// Implémentation use case
public class GetUserUseCase : IGetUserUseCase
{
    private readonly IUserRepository _userRepository;

    public GetUserUseCase(IUserRepository userRepository)
    {
        _userRepository = userRepository;
    }

    public UserDto Execute(Guid userId)
    {
        var user = _userRepository.GetById(userId);
        return new UserDto(user.Name, user.Email);
    }
}

// Contrôleur
[ApiController]
[Route("api/users")]
public class UserController : ControllerBase
{
    private readonly IGetUserUseCase _getUserUseCase;

    public UserController(IGetUserUseCase getUserUseCase)
    {
        _getUserUseCase = getUserUseCase;
    }

    [HttpGet("{id}")]
    public IActionResult GetUser(Guid id)
    {
        var result = _getUserUseCase.Execute(id);
        return Ok(result);
    }
}
```

L’injection de dépendances dans `Startup.cs` configure la résolution :

```csharp
services.AddScoped<IUserRepository, UserRepository>();
services.AddScoped<IGetUserUseCase, GetUserUseCase>();
```

---

## 2. Node.js (avec TypeScript) et architectures hexagonales

### Intégration

- Le découpage en ports (interfaces) et adaptateurs (implémentations) correspond à la Clean Architecture.  
- Frameworks comme Express ou NestJS protègent l’implémentation en isolant la logique métier.  
- Utilisation d’IOC containers (ex : `inversify`) facilite l’injection des dépendances.

### Exemple simple avec `inversify`

```typescript
// Interface
interface IUserRepository {
  getById(id: string): Promise<User>;
}

// Use case
@injectable()
class GetUserUseCase {
  constructor(@inject("IUserRepository") private userRepository: IUserRepository) {}

  async execute(id: string): Promise<UserDto> {
    const user = await this.userRepository.getById(id);
    return new UserDto(user.name, user.email);
  }
}
```

Le contrôleur (Express ou NestJS) consomme ce use case injecté.

---

## 3. Spring Boot : couches clairement délimitées avec composants Spring

### Intégration

- Spring propose annotations (`@Service`, `@Repository`, `@Controller`) pour structurer les couches.  
- L’IoC container de Spring gère l’injection des dépendances entre use cases et repositories.  
- Les tests unitaires et d’intégration sont facilités par Spring Test.

### Exemple

```java
@Service
public class GetUserUseCase {
    private final UserRepository userRepository;

    public GetUserUseCase(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public UserDto execute(Long userId) {
        User user = userRepository.findById(userId).orElseThrow();
        return new UserDto(user.getName(), user.getEmail());
    }
}

@RestController
@RequestMapping("/api/users")
public class UserController {
    private final GetUserUseCase getUserUseCase;

    public UserController(GetUserUseCase getUserUseCase) {
        this.getUserUseCase = getUserUseCase;
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserDto> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(getUserUseCase.execute(id));
    }
}
```

---

## 4. Next.js 14 : intégration côté frontend et API routes

### Intégration

- Next.js combine frontend React et API serverless via ses API Routes.  
- La logique métier (use cases) peut être isolée dans des modules communs.  
- Les API Routes jouent le rôle d’adaptateurs (interface utilisateur vers use cases).  
- Support natif pour TypeScript et serveur Node.js.

### Exemple API route

```typescript
// usecases/getUser.ts
export async function getUser(userId: string): Promise<UserDto> {
  // Accès aux données via un repository abstrait/pour mock
  const user = await userRepository.getById(userId);
  return { name: user.name, email: user.email };
}

// pages/api/user/[id].ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { getUser } from '../../../usecases/getUser';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if(req.method === 'GET') {
    const user = await getUser(req.query.id as string);
    res.status(200).json(user);
  } else {
    res.status(405).end();
  }
}
```

---

## 5. Diagramme Mermaid synthétisant l’architecture appliquée à ces frameworks

```mermaid
graph TD
    UI[Interface Utilisateur (Frontend)]
    API[API / Contrôleur]
    UC[Use Cases (Logique métier)]
    Repo[Repositories (Interfaces)]
    Infra[Infrastructure (DB, Services externes)]

    UI --> API
    API --> UC
    UC --> Repo
    Repo --> Infra

    classDef layer fill:#f0f9ff,stroke:#0366d6,stroke-width:1px
    class UI,API,UC,Repo,Infra layer
```

---

## 6. Sources et références

- Microsoft Docs, [Clean Architecture with ASP.NET Core](https://docs.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures#clean-architecture)  
- Node.js Best Practices, https://github.com/goldbergyoni/nodebestpractices#clean-architecture  
- Spring Guides, [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)  
- Next.js Documentation, https://nextjs.org/docs  
- Robert C. Martin, *Clean Architecture*, 2017  

---

Appliquer la Clean Architecture avec ces frameworks modernes permet de bénéficier à la fois des bonnes pratiques d’architecture logicielle et des avantages spécifiques de chaque environnement, assurant modularité, testabilité et évolutivité des applications.