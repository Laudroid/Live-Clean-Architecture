Voici deux approches pour aborder le TP de refactoring SOLID, présentées sous forme de chapitres distincts.

---

### Chapitre 1 : Refactoring Incrémental - Maîtrise des Fondamentaux SOLID

Cette première solution se concentre sur une application directe et incrémentale des principes SOLID, en particulier le Principe de Responsabilité Unique (SRP), le Principe Ouvert/Fermé (OCP) et le Principe d'Inversion de Dépendance (DIP). L'objectif est de décomposer le monolithe en composants plus petits et plus gérables.

#### 1. Le Code Existant (Point de Départ)

Nous allons simuler un système de gestion de produits en Python.

**Structure du code initial :**


```python
# product_manager/models.py
from dataclasses import dataclass

@dataclass
class Product:
    id: int = None
    name: str = None
    description: str = None
    price: float = None
    category: str = None

# product_manager/repository.py
from typing import List, Optional

class ProductRepository:
    """Gère la persistance des produits en mémoire."""
    def __init__(self):
        self._products: dict[int, Product] = {}
        self._next_id = 1

    def add(self, product: Product) -> Product:
        product.id = self._next_id
        self._products[self._next_id] = product
        self._next_id += 1
        return product

    def get_by_id(self, product_id: int) -> Optional[Product]:
        return self._products.get(product_id)

    def get_all(self) -> List[Product]:
        return list(self._products.values())

    def update(self, product: Product) -> bool:
        if product.id in self._products:
            self._products[product.id] = product
            return True
        return False

    def delete(self, product_id: int) -> bool:
        if product_id in self._products:
            del self._products[product_id]
            return True
        return False

# product_manager/service.py
from product_manager.models import Product
from product_manager.repository import ProductRepository

class ProductService:
    """
    Service monolithique gérant les opérations CRUD et la validation.
    Viole le SRP, l'OCP et le DIP.
    """
    def __init__(self):
        self.repository = ProductRepository() # Couplage fort

    def create_product(self, name: str, description: str, price: float, category: str) -> Product:
        if not name or not price or price <= 0:
            raise ValueError("Nom et prix sont requis, et le prix doit être positif.")
        product = Product(name=name, description=description, price=price, category=category)
        return self.repository.add(product)

    def get_product(self, product_id: int) -> Optional[Product]:
        return self.repository.get_by_id(product_id)

    def get_all_products(self) -> List[Product]:
        return self.repository.get_all()

    def update_product(self, product_id: int, name: str, description: str, price: float, category: str) -> Product:
        product = self.repository.get_by_id(product_id)
        if not product:
            raise ValueError("Produit non trouvé.")
        if not name or not price or price <= 0:
            raise ValueError("Nom et prix sont requis, et le prix doit être positif.")
        
        product.name = name
        product.description = description
        product.price = price
        product.category = category
        self.repository.update(product)
        return product

    def delete_product(self, product_id: int) -> bool:
        return self.repository.delete(product_id)

# Exemple d'utilisation (avant refactoring)
# service = ProductService()
# p1 = service.create_product("Laptop", "Puissant", 1200.0, "Électronique")
# print(service.get_all_products())
```


#### 2. Analyse et Identification des Violations SOLID

*   **SRP (Single Responsibility Principle) - Principe de Responsabilité Unique :**
    *   `ProductService` est le principal coupable. Il gère la création, la lecture, la mise à jour, la suppression, *et* la validation des produits. Il a plusieurs raisons de changer (changement de logique de validation, changement de logique CRUD, changement de persistance).
*   **OCP (Open/Closed Principle) - Principe Ouvert/Fermé :**
    *   `ProductService` n'est pas ouvert à l'extension mais fermé à la modification. Si nous voulons ajouter une nouvelle règle de validation ou changer le mécanisme de persistance, nous devrions modifier directement `ProductService`.
*   **DIP (Dependency Inversion Principle) - Principe d'Inversion de Dépendance :**
    *   `ProductService` dépend directement de l'implémentation concrète `ProductRepository` (`self.repository = ProductRepository()`). Les modules de haut niveau dépendent des modules de bas niveau.
*   **ISP (Interface Segregation Principle) - Principe de Ségrégation des Interfaces :**
    *   Non applicable directement car aucune interface n'est encore définie. Cependant, si `ProductRepository` était une interface, elle pourrait devenir "grasse" si elle contenait des méthodes que tous les clients n'utilisent pas.
*   **LSP (Liskov Substitution Principle) - Principe de Substitution de Liskov :**
    *   Non directement violé dans ce code simple, car il n'y a pas de hiérarchie de classes complexe avec des sous-types.

#### 3. Application du Principe de Responsabilité Unique (SRP)

Nous allons décomposer `ProductService` en plusieurs entités, chacune avec une responsabilité unique.


```python
# product_manager/interfaces.py
from abc import ABC, abstractmethod
from typing import List, Optional
from product_manager.models import Product

class IProductRepository(ABC):
    """Interface pour les opérations de persistance des produits."""
    @abstractmethod
    def add(self, product: Product) -> Product: pass
    @abstractmethod
    def get_by_id(self, product_id: int) -> Optional[Product]: pass
    @abstractmethod
    def get_all(self) -> List[Product]: pass
    @abstractmethod
    def update(self, product: Product) -> bool: pass
    @abstractmethod
    def delete(self, product_id: int) -> bool: pass

class IProductValidator(ABC):
    """Interface pour la validation des produits."""
    @abstractmethod
    def validate(self, product: Product): pass

# product_manager/validators.py
from product_manager.models import Product
from product_manager.interfaces import IProductValidator

class BasicProductValidator(IProductValidator):
    """Implémentation de base de la validation des produits."""
    def validate(self, product: Product):
        if not product.name or not product.price or product.price <= 0:
            raise ValueError("Nom et prix sont requis, et le prix doit être positif.")

# product_manager/repository.py (modifié pour implémenter l'interface)
# ... (imports existants) ...
from product_manager.interfaces import IProductRepository

class InMemoryProductRepository(IProductRepository): # Renommé pour clarté
    """Gère la persistance des produits en mémoire, implémente IProductRepository."""
    def __init__(self):
        self._products: dict[int, Product] = {}
        self._next_id = 1

    def add(self, product: Product) -> Product:
        product.id = self._next_id
        self._products[self._next_id] = product
        self._next_id += 1
        return product

    def get_by_id(self, product_id: int) -> Optional[Product]:
        return self._products.get(product_id)

    def get_all(self) -> List[Product]:
        return list(self._products.values())

    def update(self, product: Product) -> bool:
        if product.id in self._products:
            self._products[product.id] = product
            return True
        return False

    def delete(self, product_id: int) -> bool:
        if product_id in self._products:
            del self._products[product_id]
            return True
        return False

# product_manager/services.py (nouveaux services spécifiques)
from product_manager.models import Product
from product_manager.interfaces import IProductRepository, IProductValidator
from typing import List, Optional

class ProductCreator:
    """Responsabilité unique : créer un produit."""
    def __init__(self, repository: IProductRepository, validator: IProductValidator):
        self.repository = repository
        self.validator = validator

    def create(self, name: str, description: str, price: float, category: str) -> Product:
        product = Product(name=name, description=description, price=price, category=category)
        self.validator.validate(product)
        return self.repository.add(product)

class ProductRetriever:
    """Responsabilité unique : récupérer des produits."""
    def __init__(self, repository: IProductRepository):
        self.repository = repository

    def get_by_id(self, product_id: int) -> Optional[Product]:
        return self.repository.get_by_id(product_id)

    def get_all(self) -> List[Product]:
        return self.repository.get_all()

class ProductUpdater:
    """Responsabilité unique : mettre à jour un produit."""
    def __init__(self, repository: IProductRepository, validator: IProductValidator):
        self.repository = repository
        self.validator = validator

    def update(self, product_id: int, name: str, description: str, price: float, category: str) -> Product:
        product = self.repository.get_by_id(product_id)
        if not product:
            raise ValueError("Produit non trouvé.")
        
        updated_product = Product(id=product_id, name=name, description=description, price=price, category=category)
        self.validator.validate(updated_product) # Valide les nouvelles données
        
        # Mettre à jour l'objet existant ou créer un nouveau si l'entité est immuable
        product.name = name
        product.description = description
        product.price = price
        product.category = category
        self.repository.update(product)
        return product

class ProductDeleter:
    """Responsabilité unique : supprimer un produit."""
    def __init__(self, repository: IProductRepository):
        self.repository = repository

    def delete(self, product_id: int) -> bool:
        return self.repository.delete(product_id)
```


#### 4. Application du Principe Ouvert/Fermé (OCP) et d'Inversion des Dépendances (DIP)

*   **DIP :** Les nouveaux services (`ProductCreator`, etc.) dépendent désormais des interfaces (`IProductRepository`, `IProductValidator`) et non des implémentations concrètes. Les implémentations sont injectées via le constructeur.
*   **OCP :**
    *   Pour changer de mécanisme de persistance (ex: passer à une base de données), il suffit de créer une nouvelle classe implémentant `IProductRepository` sans modifier les services existants.
    *   Pour ajouter une nouvelle règle de validation, on peut créer une nouvelle implémentation de `IProductValidator` ou composer plusieurs validateurs.

#### 5. Application du Principe de Ségrégation des Interfaces (ISP) et de Substitution de Liskov (LSP)

*   **ISP :** L'interface `IProductRepository` est encore assez "générique" pour les opérations CRUD. Pour une ségrégation plus fine, on pourrait la diviser en `IProductReader` (pour `get_by_id`, `get_all`) et `IProductWriter` (pour `add`, `update`, `delete`). Cela permettrait à un client qui n'a besoin que de lire de ne dépendre que de `IProductReader`. Pour ce TP, l'interface unique est un bon compromis initial.
*   **LSP :** En utilisant des interfaces et en injectant des dépendances, nous nous assurons que toute implémentation de `IProductRepository` ou `IProductValidator` peut être substituée à son type de base sans casser le programme, respectant ainsi le LSP.

#### 6. Implémentation d'une Nouvelle Fonctionnalité (Recherche)

La nouvelle fonctionnalité "Recherche de produits par catégorie et par prix maximum" sera implémentée comme un nouveau service, respectant les principes SOLID.


```python
# product_manager/services.py (ajout)
# ... (imports existants) ...

class ProductSearcher:
    """Responsabilité unique : rechercher des produits."""
    def __init__(self, repository: IProductRepository):
        self.repository = repository

    def search_by_category_and_max_price(self, category: str, max_price: float) -> List[Product]:
        if max_price <= 0:
            raise ValueError("Le prix maximum doit être positif.")
        
        all_products = self.repository.get_all()
        filtered_products = [
            product for product in all_products
            if product.category == category and product.price <= max_price
        ]
        return filtered_products

# product_manager/main.py (Exemple d'assemblage et d'utilisation)
from product_manager.models import Product
from product_manager.repository import InMemoryProductRepository
from product_manager.validators import BasicProductValidator
from product_manager.services import ProductCreator, ProductRetriever, ProductUpdater, ProductDeleter, ProductSearcher

# Assemblage des dépendances (peut être fait par un conteneur DI dans une application réelle)
product_repo = InMemoryProductRepository()
product_validator = BasicProductValidator()

product_creator = ProductCreator(product_repo, product_validator)
product_retriever = ProductRetriever(product_repo)
product_updater = ProductUpdater(product_repo, product_validator)
product_deleter = ProductDeleter(product_repo)
product_searcher = ProductSearcher(product_repo)

# Utilisation des services refactorisés
p1 = product_creator.create("Laptop", "Puissant", 1200.0, "Électronique")
p2 = product_creator.create("Souris", "Ergonomique", 25.0, "Accessoires")
p3 = product_creator.create("Clavier", "Mécanique", 100.0, "Accessoires")
p4 = product_creator.create("Smartphone", "Dernière génération", 800.0, "Électronique")

print("Tous les produits:", product_retriever.get_all())

# Nouvelle fonctionnalité
electronic_products_under_1000 = product_searcher.search_by_category_and_max_price("Électronique", 1000.0)
print("Produits électroniques de moins de 1000€:", electronic_products_under_1000)

# Mise à jour
product_updater.update(p1.id, "Laptop Pro", "Très puissant", 1500.0, "Électronique")
print("Produit mis à jour:", product_retriever.get_by_id(p1.id))

# Suppression
product_deleter.delete(p2.id)
print("Tous les produits après suppression:", product_retriever.get_all())
```


Cette approche a permis de créer un code plus modulaire, testable et extensible. Chaque composant a une raison unique de changer, et les dépendances sont gérées via des abstractions.

---

### Chapitre 2 : Refactoring Avancé - Architecture par Cas d'Utilisation (Use Cases) et Command/Query

Cette deuxième solution va plus loin en introduisant une architecture par cas d'utilisation (Use Cases) et le pattern Command/Query. Cela permet une séparation encore plus nette des préoccupations, en isolant la logique métier pure des détails d'implémentation (persistance, validation, présentation).

#### 1. Le Code Existant (Point de Départ)

Nous partons du même code monolithique initial que dans le Chapitre 1.

#### 2. Analyse et Identification des Violations SOLID

Les violations sont les mêmes que celles identifiées précédemment, mais nous allons les adresser avec une structure plus formelle inspirée de la Clean Architecture.

#### 3. Refactoring avec une Architecture par Cas d'Utilisation

L'idée est de structurer l'application autour de "cas d'utilisation" (ou "interacteurs"), qui encapsulent une logique métier spécifique.

**Structure des dossiers proposée :**


```
product_manager_advanced/
├── domain/
│   └── entities.py           # Entités de domaine pures
├── application/
│   ├── interfaces/
│   │   └── repositories.py   # Interfaces pour la persistance
│   │   └── validators.py     # Interfaces pour la validation
│   ├── use_cases/            # Logique métier spécifique à l'application
│   │   ├── commands/
│   │   └── queries/
│   └── dtos.py               # Data Transfer Objects pour les entrées/sorties des Use Cases
├── infrastructure/
│   ├── persistence/
│   │   └── in_memory_repository.py # Implémentation concrète du repository
│   └── validation/
│       └── basic_validator.py      # Implémentation concrète du validateur
├── presentation/             # Point d'entrée de l'application (ex: CLI, API)
│   └── cli.py
└── main.py                   # Assemblage des dépendances
```


**Implémentation des composants :**


```python
# product_manager_advanced/domain/entities.py
from dataclasses import dataclass

@dataclass(frozen=True) # Rendre les entités immuables est une bonne pratique
class Product:
    id: int = None
    name: str = None
    description: str = None
    price: float = None
    category: str = None

# product_manager_advanced/application/dtos.py
from dataclasses import dataclass
from typing import Optional

@dataclass(frozen=True)
class CreateProductRequest:
    name: str
    description: Optional[str]
    price: float
    category: str

@dataclass(frozen=True)
class UpdateProductRequest:
    id: int
    name: str
    description: Optional[str]
    price: float
    category: str

@dataclass(frozen=True)
class ProductResponse: # Pour les sorties des Use Cases
    id: int
    name: str
    description: Optional[str]
    price: float
    category: str

@dataclass(frozen=True)
class SearchProductsRequest:
    category: str
    max_price: float

# product_manager_advanced/application/interfaces/repositories.py
from abc import ABC, abstractmethod
from typing import List, Optional
from product_manager_advanced.domain.entities import Product

class IProductRepository(ABC):
    """Interface pour les opérations de persistance des produits."""
    @abstractmethod
    def add(self, product: Product) -> Product: pass
    @abstractmethod
    def get_by_id(self, product_id: int) -> Optional[Product]: pass
    @abstractmethod
    def get_all(self) -> List[Product]: pass
    @abstractmethod
    def update(self, product: Product) -> bool: pass
    @abstractmethod
    def delete(self, product_id: int) -> bool: pass

# product_manager_advanced/application/interfaces/validators.py
from abc import ABC, abstractmethod
from product_manager_advanced.domain.entities import Product

class IProductValidator(ABC):
    """Interface pour la validation des produits."""
    @abstractmethod
    def validate(self, product: Product): pass

# product_manager_advanced/infrastructure/persistence/in_memory_repository.py
from product_manager_advanced.domain.entities import Product
from product_manager_advanced.application.interfaces.repositories import IProductRepository
from typing import List, Optional

class InMemoryProductRepository(IProductRepository):
    """Implémentation en mémoire de IProductRepository."""
    def __init__(self):
        self._products: dict[int, Product] = {}
        self._next_id = 1

    def add(self, product: Product) -> Product:
        new_id = self._next_id
        new_product = Product(id=new_id, name=product.name, description=product.description, price=product.price, category=product.category)
        self._products[new_id] = new_product
        self._next_id += 1
        return new_product

    def get_by_id(self, product_id: int) -> Optional[Product]:
        return self._products.get(product_id)

    def get_all(self) -> List[Product]:
        return list(self._products.values())

    def update(self, product: Product) -> bool:
        if product.id in self._products:
            self._products[product.id] = product
            return True
        return False

    def delete(self, product_id: int) -> bool:
        if product_id in self._products:
            del self._products[product_id]
            return True
        return False

# product_manager_advanced/infrastructure/validation/basic_validator.py
from product_manager_advanced.domain.entities import Product
from product_manager_advanced.application.interfaces.validators import IProductValidator

class BasicProductValidator(IProductValidator):
    """Implémentation de base de la validation des produits."""
    def validate(self, product: Product):
        if not product.name or not product.price or product.price <= 0:
            raise ValueError("Nom et prix sont requis, et le prix doit être positif.")

# product_manager_advanced/application/use_cases/commands.py
from product_manager_advanced.domain.entities import Product
from product_manager_advanced.application.interfaces.repositories import IProductRepository
from product_manager_advanced.application.interfaces.validators import IProductValidator
from product_manager_advanced.application.dtos import CreateProductRequest, UpdateProductRequest, ProductResponse

class CreateProductCommand:
    """Cas d'utilisation pour la création d'un produit."""
    def __init__(self, repository: IProductRepository, validator: IProductValidator):
        self.repository = repository
        self.validator = validator

    def execute(self, request: CreateProductRequest) -> ProductResponse:
        product = Product(name=request.name, description=request.description, price=request.price, category=request.category)
        self.validator.validate(product)
        created_product = self.repository.add(product)
        return ProductResponse(id=created_product.id, name=created_product.name, description=created_product.description, price=created_product.price, category=created_product.category)

class UpdateProductCommand:
    """Cas d'utilisation pour la mise à jour d'un produit."""
    def __init__(self, repository: IProductRepository, validator: IProductValidator):
        self.repository = repository
        self.validator = validator

    def execute(self, request: UpdateProductRequest) -> ProductResponse:
        existing_product = self.repository.get_by_id(request.id)
        if not existing_product:
            raise ValueError(f"Produit avec l'ID {request.id} non trouvé.")
        
        updated_product_entity = Product(id=request.id, name=request.name, description=request.description, price=request.price, category=request.category)
        self.validator.validate(updated_product_entity)
        
        self.repository.update(updated_product_entity)
        return ProductResponse(id=updated_product_entity.id, name=updated_product_entity.name, description=updated_product_entity.description, price=updated_product_entity.price, category=updated_product_entity.category)

class DeleteProductCommand:
    """Cas d'utilisation pour la suppression d'un produit."""
    def __init__(self, repository: IProductRepository):
        self.repository = repository

    def execute(self, product_id: int) -> bool:
        return self.repository.delete(product_id)

# product_manager_advanced/application/use_cases/queries.py
from product_manager_advanced.domain.entities import Product
from product_manager_advanced.application.interfaces.repositories import IProductRepository
from product_manager_advanced.application.dtos import ProductResponse, SearchProductsRequest
from typing import List, Optional

class GetProductQuery:
    """Cas d'utilisation pour récupérer un produit par ID."""
    def __init__(self, repository: IProductRepository):
        self.repository = repository

    def execute(self, product_id: int) -> Optional[ProductResponse]:
        product = self.repository.get_by_id(product_id)
        if product:
            return ProductResponse(id=product.id, name=product.name, description=product.description, price=product.price, category=product.category)
        return None

class GetAllProductsQuery:
    """Cas d'utilisation pour récupérer tous les produits."""
    def __init__(self, repository: IProductRepository):
        self.repository = repository

    def execute(self) -> List[ProductResponse]:
        products = self.repository.get_all()
        return [ProductResponse(id=p.id, name=p.name, description=p.description, price=p.price, category=p.category) for p in products]

# product_manager_advanced/application/use_cases/queries.py (ajout de la nouvelle fonctionnalité)
class SearchProductsQuery:
    """Cas d'utilisation pour rechercher des produits par catégorie et prix maximum."""
    def __init__(self, repository: IProductRepository):
        self.repository = repository

    def execute(self, request: SearchProductsRequest) -> List[ProductResponse]:
        if request.max_price <= 0:
            raise ValueError("Le prix maximum doit être positif.")
        
        all_products = self.repository.get_all()
        filtered_products = [
            product for product in all_products
            if product.category == request.category and product.price <= request.max_price
        ]
        return [ProductResponse(id=p.id, name=p.name, description=p.description, price=p.price, category=p.category) for p in filtered_products]

# product_manager_advanced/main.py (Assemblage et utilisation)
from product_manager_advanced.infrastructure.persistence.in_memory_repository import InMemoryProductRepository
from product_manager_advanced.infrastructure.validation.basic_validator import BasicProductValidator
from product_manager_advanced.application.use_cases.commands import CreateProductCommand, UpdateProductCommand, DeleteProductCommand
from product_manager_advanced.application.use_cases.queries import GetProductQuery, GetAllProductsQuery, SearchProductsQuery
from product_manager_advanced.application.dtos import CreateProductRequest, UpdateProductRequest, SearchProductsRequest

# Assemblage des dépendances
product_repo = InMemoryProductRepository()
product_validator = BasicProductValidator()

# Instanciation des cas d'utilisation
create_product_command = CreateProductCommand(product_repo, product_validator)
update_product_command = UpdateProductCommand(product_repo, product_validator)
delete_product_command = DeleteProductCommand(product_repo)
get_product_query = GetProductQuery(product_repo)
get_all_products_query = GetAllProductsQuery(product_repo)
search_products_query = SearchProductsQuery(product_repo)

# Utilisation des cas d'utilisation
p1_request = CreateProductRequest(name="Laptop", description="Puissant", price=1200.0, category="Électronique")
p1_response = create_product_command.execute(p1_request)
print(f"Produit créé: {p1_response}")

p2_request = CreateProductRequest(name="Souris", description="Ergonomique", price=25.0, category="Accessoires")
p2_response = create_product_command.execute(p2_request)
print(f"Produit créé: {p2_response}")

p3_request = CreateProductRequest(name="Clavier", description="Mécanique", price=100.0, category="Accessoires")
p3_response = create_product_command.execute(p3_request)
print(f"Produit créé: {p3_response}")

all_products = get_all_products_query.execute()
print("\nTous les produits:", all_products)

# Nouvelle fonctionnalité (recherche)
search_request = SearchProductsRequest(category="Électronique", max_price=1000.0)
electronic_products_under_1000 = search_products_query.execute(search_request)
print("\nProduits électroniques de moins de 1000€:", electronic_products_under_1000)

# Mise à jour
update_request = UpdateProductRequest(id=p1_response.id, name="Laptop Pro", description="Très puissant", price=1500.0, category="Électronique")
updated_p1 = update_product_command.execute(update_request)
print(f"\nProduit mis à jour: {updated_p1}")

# Suppression
delete_product_command.execute(p2_response.id)
print("\nTous les produits après suppression:", get_all_products_query.execute())
```


#### 4. Avantages de cette Approche

*   **SRP renforcé :** Chaque cas d'utilisation a une seule raison de changer (une action métier spécifique).
*   **OCP amélioré :** L'ajout d'une nouvelle fonctionnalité métier signifie généralement l'ajout d'un nouveau cas d'utilisation, sans modifier les existants.
*   **DIP omniprésent :** Tous les cas d'utilisation dépendent d'interfaces (repositories, validateurs), et non d'implémentations concrètes.
*   **ISP :** Les interfaces de repository pourraient être affinées (ex: `IProductReader`, `IProductWriter`) si certains cas d'utilisation n'ont besoin que d'un sous-ensemble des opérations.
*   **LSP :** L'utilisation d'interfaces et l'injection de dépendances garantissent que les implémentations peuvent être substituées sans problème.
*   **Testabilité accrue :** Chaque cas d'utilisation peut être testé isolément en mockant ses dépendances (interfaces).
*   **Clarté métier :** Les cas d'utilisation reflètent directement les actions que l'application peut effectuer, rendant la logique métier plus facile à comprendre et à maintenir.
*   **Indépendance des couches :** La couche `application` (use cases) est indépendante des détails d'infrastructure (`persistence`, `validation`) et de présentation.

Cette approche, bien que plus verbeuse au départ, offre une flexibilité et une maintenabilité exceptionnelles pour les systèmes complexes et évolutifs. Elle prépare l'application à des changements majeurs (ex: changement de base de données, ajout d'une API REST, d'une interface graphique) avec un impact minimal sur la logique métier centrale.