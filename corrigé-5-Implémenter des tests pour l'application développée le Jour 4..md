Voici deux solutions pour le TP sur les tests et les stratégies de déploiement d'une application Clean Architecture, en utilisant Python pour les exemples de code.

---

### Chapitre 1 : Tests Unitaires et d'Intégration Fondamentaux & Stratégies de Déploiement Classiques

Cette solution se concentre sur l'implémentation des tests essentiels pour le cœur de l'application et ses adaptateurs, puis explore les stratégies de déploiement courantes adaptées à une architecture propre.

#### Code de l'Application

Pour ce chapitre, nous allons utiliser une structure de code Python simple pour notre application de gestion de catalogue de produits.

**Structure du projet :**


```
product_catalog/
├── domain/
│   ├── entities/
│   │   └── product.py
│   └── ports/
│       ├── product_repository.py
│       └── product_output_port.py
├── application/
│   └── use_cases/
│       ├── create_product_use_case.py
│       ├── get_product_by_id_use_case.py
│       ├── update_product_use_case.py
│       ├── delete_product_use_case.py
│       └── get_all_products_use_case.py
└── infrastructure/
    ├── adapters/
    │   └── in_memory_product_repository.py
    └── main.py
```


**`product_catalog/domain/entities/product.py`**


```python
import uuid
from decimal import Decimal

class Product:
    def __init__(self, name: str, description: str, price: Decimal, stock: int, id: str = None):
        if not name:
            raise ValueError("Le nom du produit ne peut pas être vide.")
        if price <= 0:
            raise ValueError("Le prix du produit doit être strictement positif.")
        if stock < 0:
            raise ValueError("Le stock du produit ne peut pas être négatif.")

        self.id = id if id else str(uuid.uuid4())
        self.name = name
        self.description = description
        self.price = price
        self.stock = stock

    def update_name(self, new_name: str):
        if not new_name:
            raise ValueError("Le nom du produit ne peut pas être vide.")
        self.name = new_name

    def update_description(self, new_description: str):
        self.description = new_description

    def change_price(self, new_price: Decimal):
        if new_price <= 0:
            raise ValueError("Le prix du produit doit être strictement positif.")
        self.price = new_price

    def adjust_stock(self, quantity: int):
        if self.stock + quantity < 0:
            raise ValueError("Le stock ne peut pas devenir négatif.")
        self.stock += quantity

    def __eq__(self, other):
        if not isinstance(other, Product):
            return NotImplemented
        return self.id == other.id

    def __hash__(self):
        return hash(self.id)
```


**`product_catalog/domain/ports/product_repository.py`**


```python
from abc import ABC, abstractmethod
from typing import List, Optional
from product_catalog.domain.entities.product import Product

class ProductRepository(ABC):
    @abstractmethod
    def save(self, product: Product) -> Product: pass
    @abstractmethod
    def find_by_id(self, product_id: str) -> Optional[Product]: pass
    @abstractmethod
    def find_all(self) -> List[Product]: pass
    @abstractmethod
    def update(self, product: Product) -> Product: pass
    @abstractmethod
    def delete(self, product_id: str) -> None: pass
```


**`product_catalog/domain/ports/product_output_port.py`**


```python
from abc import ABC, abstractmethod
from typing import List
from product_catalog.domain.entities.product import Product

class ProductOutputPort(ABC):
    @abstractmethod
    def present_product(self, product: Product): pass
    @abstractmethod
    def present_products(self, products: List[Product]): pass
    @abstractmethod
    def present_error(self, message: str): pass
```


**`product_catalog/application/use_cases/create_product_use_case.py`**


```python
from decimal import Decimal
from product_catalog.domain.entities.product import Product
from product_catalog.domain.ports.product_repository import ProductRepository
from product_catalog.domain.ports.product_output_port import ProductOutputPort

class CreateProductUseCase:
    def __init__(self, repository: ProductRepository, presenter: ProductOutputPort):
        self.repository = repository
        self.presenter = presenter

    def execute(self, name: str, description: str, price: Decimal, stock: int):
        try:
            product = Product(name, description, price, stock)
            self.repository.save(product)
            self.presenter.present_product(product)
        except ValueError as e:
            self.presenter.present_error(str(e))
```


*(Les autres Use Cases sont similaires, utilisant le repository et le presenter pour leurs opérations et la gestion des erreurs.)*

**`product_catalog/application/use_cases/get_product_by_id_use_case.py`**


```python
from product_catalog.domain.ports.product_repository import ProductRepository
from product_catalog.domain.ports.product_output_port import ProductOutputPort

class GetProductByIdUseCase:
    def __init__(self, repository: ProductRepository, presenter: ProductOutputPort):
        self.repository = repository
        self.presenter = presenter

    def execute(self, product_id: str):
        product = self.repository.find_by_id(product_id)
        if product:
            self.presenter.present_product(product)
        else:
            self.presenter.present_error(f"Produit avec l'ID {product_id} non trouvé.")
```


**`product_catalog/application/use_cases/update_product_use_case.py`**


```python
from decimal import Decimal
from product_catalog.domain.entities.product import Product
from product_catalog.domain.ports.product_repository import ProductRepository
from product_catalog.domain.ports.product_output_port import ProductOutputPort

class UpdateProductUseCase:
    def __init__(self, repository: ProductRepository, presenter: ProductOutputPort):
        self.repository = repository
        self.presenter = presenter

    def execute(self, product_id: str, name: str, description: str, price: Decimal, stock: int):
        try:
            product = self.repository.find_by_id(product_id)
            if not product:
                self.presenter.present_error(f"Produit avec l'ID {product_id} non trouvé.")
                return

            # Appliquer les mises à jour via les méthodes de l'entité
            product.update_name(name)
            product.update_description(description)
            product.change_price(price)
            product.adjust_stock(stock - product.stock) # Calculer la différence pour adjust_stock

            updated_product = self.repository.update(product)
            self.presenter.present_product(updated_product)
        except ValueError as e:
            self.presenter.present_error(str(e))
```


**`product_catalog/application/use_cases/delete_product_use_case.py`**


```python
from product_catalog.domain.ports.product_repository import ProductRepository
from product_catalog.domain.ports.product_output_port import ProductOutputPort

class DeleteProductUseCase:
    def __init__(self, repository: ProductRepository, presenter: ProductOutputPort):
        self.repository = repository
        self.presenter = presenter

    def execute(self, product_id: str):
        product = self.repository.find_by_id(product_id)
        if not product:
            self.presenter.present_error(f"Produit avec l'ID {product_id} non trouvé.")
            return
        self.repository.delete(product_id)
        self.presenter.present_product(product) # Ou un message de succès
```


**`product_catalog/application/use_cases/get_all_products_use_case.py`**


```python
from product_catalog.domain.ports.product_repository import ProductRepository
from product_catalog.domain.ports.product_output_port import ProductOutputPort

class GetAllProductsUseCase:
    def __init__(self, repository: ProductRepository, presenter: ProductOutputPort):
        self.repository = repository
        self.presenter = presenter

    def execute(self):
        products = self.repository.find_all()
        self.presenter.present_products(products)
```


**`product_catalog/infrastructure/adapters/in_memory_product_repository.py`**


```python
from typing import List, Optional, Dict
from product_catalog.domain.entities.product import Product
from product_catalog.domain.ports.product_repository import ProductRepository

class InMemoryProductRepository(ProductRepository):
    def __init__(self):
        self._products: Dict[str, Product] = {}

    def save(self, product: Product) -> Product:
        self._products[product.id] = product
        return product

    def find_by_id(self, product_id: str) -> Optional[Product]:
        return self._products.get(product_id)

    def find_all(self) -> List[Product]:
        return list(self._products.values())

    def update(self, product: Product) -> Product:
        if product.id not in self._products:
            raise ValueError(f"Produit avec l'ID {product.id} non trouvé pour la mise à jour.")
        self._products[product.id] = product
        return product

    def delete(self, product_id: str) -> None:
        if product_id in self._products:
            del self._products[product_id]
        else:
            raise ValueError(f"Produit avec l'ID {product_id} non trouvé pour la suppression.")
```


#### Partie 1 : Tests Unitaires (Core Domain)

**Fichier de tests : `tests/unit/test_domain.py`**


```python
import pytest
from decimal import Decimal
from product_catalog.domain.entities.product import Product
from unittest.mock import Mock

# 1.1. Tests des Entités (Product)
def test_product_creation_valid():
    product = Product("Laptop", "Puissant", Decimal("1200.00"), 10)
    assert product.name == "Laptop"
    assert product.price == Decimal("1200.00")
    assert product.stock == 10
    assert product.id is not None

def test_product_creation_invalid_name():
    with pytest.raises(ValueError, match="Le nom du produit ne peut pas être vide."):
        Product("", "Description", Decimal("10.00"), 5)

def test_product_creation_invalid_price():
    with pytest.raises(ValueError, match="Le prix du produit doit être strictement positif."):
        Product("Test", "Description", Decimal("0.00"), 5)
    with pytest.raises(ValueError, match="Le prix du produit doit être strictement positif."):
        Product("Test", "Description", Decimal("-10.00"), 5)

def test_product_creation_invalid_stock():
    with pytest.raises(ValueError, match="Le stock du produit ne peut pas être négatif."):
        Product("Test", "Description", Decimal("10.00"), -1)

def test_product_update_name():
    product = Product("Old Name", "Desc", Decimal("100"), 5)
    product.update_name("New Name")
    assert product.name == "New Name"

def test_product_update_name_empty():
    product = Product("Old Name", "Desc", Decimal("100"), 5)
    with pytest.raises(ValueError, match="Le nom du produit ne peut pas être vide."):
        product.update_name("")

def test_product_change_price():
    product = Product("Test", "Desc", Decimal("100"), 5)
    product.change_price(Decimal("150.00"))
    assert product.price == Decimal("150.00")

def test_product_change_price_invalid():
    product = Product("Test", "Desc", Decimal("100"), 5)
    with pytest.raises(ValueError, match="Le prix du produit doit être strictement positif."):
        product.change_price(Decimal("0.00"))

def test_product_adjust_stock():
    product = Product("Test", "Desc", Decimal("100"), 5)
    product.adjust_stock(3)
    assert product.stock == 8
    product.adjust_stock(-2)
    assert product.stock == 6

def test_product_adjust_stock_negative_result():
    product = Product("Test", "Desc", Decimal("100"), 5)
    with pytest.raises(ValueError, match="Le stock ne peut pas devenir négatif."):
        product.adjust_stock(-6)

# 1.2. Tests des Cas d'Utilisation (Use Cases)
# Mock des dépendances
@pytest.fixture
def mock_repository():
    return Mock(spec=ProductRepository)

@pytest.fixture
def mock_presenter():
    return Mock(spec=ProductOutputPort)

# Test CreateProductUseCase
from product_catalog.application.use_cases.create_product_use_case import CreateProductUseCase

def test_create_product_success(mock_repository, mock_presenter):
    use_case = CreateProductUseCase(mock_repository, mock_presenter)
    name, desc, price, stock = "Laptop", "Puissant", Decimal("1200.00"), 10
    
    use_case.execute(name, desc, price, stock)
    
    # Vérifier que le repository.save a été appelé avec un produit valide
    mock_repository.save.assert_called_once()
    saved_product = mock_repository.save.call_args[0][0]
    assert saved_product.name == name
    assert saved_product.price == price
    assert saved_product.stock == stock
    
    # Vérifier que le presenter.present_product a été appelé
    mock_presenter.present_product.assert_called_once_with(saved_product)
    mock_presenter.present_error.assert_not_called()

def test_create_product_invalid_data(mock_repository, mock_presenter):
    use_case = CreateProductUseCase(mock_repository, mock_presenter)
    name, desc, price, stock = "", "Puissant", Decimal("1200.00"), 10
    
    use_case.execute(name, desc, price, stock)
    
    # Vérifier que le repository.save n'a pas été appelé
    mock_repository.save.assert_not_called()
    
    # Vérifier que le presenter.present_error a été appelé
    mock_presenter.present_error.assert_called_once_with("Le nom du produit ne peut pas être vide.")
    mock_presenter.present_product.assert_not_called()

# Test GetProductByIdUseCase
from product_catalog.application.use_cases.get_product_by_id_use_case import GetProductByIdUseCase

def test_get_product_by_id_success(mock_repository, mock_presenter):
    use_case = GetProductByIdUseCase(mock_repository, mock_presenter)
    product_id = "123"
    expected_product = Product("Test", "Desc", Decimal("100"), 5, id=product_id)
    mock_repository.find_by_id.return_value = expected_product
    
    use_case.execute(product_id)
    
    mock_repository.find_by_id.assert_called_once_with(product_id)
    mock_presenter.present_product.assert_called_once_with(expected_product)
    mock_presenter.present_error.assert_not_called()

def test_get_product_by_id_not_found(mock_repository, mock_presenter):
    use_case = GetProductByIdUseCase(mock_repository, mock_presenter)
    product_id = "non_existent_id"
    mock_repository.find_by_id.return_value = None
    
    use_case.execute(product_id)
    
    mock_repository.find_by_id.assert_called_once_with(product_id)
    mock_presenter.present_error.assert_called_once_with(f"Produit avec l'ID {product_id} non trouvé.")
    mock_presenter.present_product.assert_not_called()

# Test UpdateProductUseCase
from product_catalog.application.use_cases.update_product_use_case import UpdateProductUseCase

def test_update_product_success(mock_repository, mock_presenter):
    use_case = UpdateProductUseCase(mock_repository, mock_presenter)
    product_id = "123"
    existing_product = Product("Old Name", "Old Desc", Decimal("100.00"), 5, id=product_id)
    mock_repository.find_by_id.return_value = existing_product
    mock_repository.update.return_value = existing_product # Simule la mise à jour
    
    new_name, new_desc, new_price, new_stock = "New Name", "New Desc", Decimal("150.00"), 8
    use_case.execute(product_id, new_name, new_desc, new_price, new_stock)
    
    mock_repository.find_by_id.assert_called_once_with(product_id)
    mock_repository.update.assert_called_once()
    
    updated_product_arg = mock_repository.update.call_args[0][0]
    assert updated_product_arg.name == new_name
    assert updated_product_arg.description == new_desc
    assert updated_product_arg.price == new_price
    assert updated_product_arg.stock == new_stock
    
    mock_presenter.present_product.assert_called_once_with(updated_product_arg)
    mock_presenter.present_error.assert_not_called()

def test_update_product_not_found(mock_repository, mock_presenter):
    use_case = UpdateProductUseCase(mock_repository, mock_presenter)
    product_id = "non_existent_id"
    mock_repository.find_by_id.return_value = None
    
    use_case.execute(product_id, "N", "D", Decimal("10"), 1)
    
    mock_repository.find_by_id.assert_called_once_with(product_id)
    mock_repository.update.assert_not_called()
    mock_presenter.present_error.assert_called_once_with(f"Produit avec l'ID {product_id} non trouvé.")
    mock_presenter.present_product.assert_not_called()

def test_update_product_invalid_data(mock_repository, mock_presenter):
    use_case = UpdateProductUseCase(mock_repository, mock_presenter)
    product_id = "123"
    existing_product = Product("Old Name", "Old Desc", Decimal("100.00"), 5, id=product_id)
    mock_repository.find_by_id.return_value = existing_product
    
    use_case.execute(product_id, "", "New Desc", Decimal("150.00"), 8) # Nom vide
    
    mock_repository.find_by_id.assert_called_once_with(product_id)
    mock_repository.update.assert_not_called()
    mock_presenter.present_error.assert_called_once_with("Le nom du produit ne peut pas être vide.")
    mock_presenter.present_product.assert_not_called()

# Test DeleteProductUseCase
from product_catalog.application.use_cases.delete_product_use_case import DeleteProductUseCase

def test_delete_product_success(mock_repository, mock_presenter):
    use_case = DeleteProductUseCase(mock_repository, mock_presenter)
    product_id = "123"
    existing_product = Product("Test", "Desc", Decimal("100"), 5, id=product_id)
    mock_repository.find_by_id.return_value = existing_product
    
    use_case.execute(product_id)
    
    mock_repository.find_by_id.assert_called_once_with(product_id)
    mock_repository.delete.assert_called_once_with(product_id)
    mock_presenter.present_product.assert_called_once_with(existing_product) # Ou un message de succès
    mock_presenter.present_error.assert_not_called()

def test_delete_product_not_found(mock_repository, mock_presenter):
    use_case = DeleteProductUseCase(mock_repository, mock_presenter)
    product_id = "non_existent_id"
    mock_repository.find_by_id.return_value = None
    
    use_case.execute(product_id)
    
    mock_repository.find_by_id.assert_called_once_with(product_id)
    mock_repository.delete.assert_not_called()
    mock_presenter.present_error.assert_called_once_with(f"Produit avec l'ID {product_id} non trouvé.")
    mock_presenter.present_product.assert_not_called()

# Test GetAllProductsUseCase
from product_catalog.application.use_cases.get_all_products_use_case import GetAllProductsUseCase

def test_get_all_products_success(mock_repository, mock_presenter):
    use_case = GetAllProductsUseCase(mock_repository, mock_presenter)
    products = [
        Product("P1", "D1", Decimal("10"), 1, id="1"),
        Product("P2", "D2", Decimal("20"), 2, id="2")
    ]
    mock_repository.find_all.return_value = products
    
    use_case.execute()
    
    mock_repository.find_all.assert_called_once()
    mock_presenter.present_products.assert_called_once_with(products)
    mock_presenter.present_error.assert_not_called()

def test_get_all_products_empty(mock_repository, mock_presenter):
    use_case = GetAllProductsUseCase(mock_repository, mock_presenter)
    mock_repository.find_all.return_value = []
    
    use_case.execute()
    
    mock_repository.find_all.assert_called_once()
    mock_presenter.present_products.assert_called_once_with([])
    mock_presenter.present_error.assert_not_called()
```


#### Partie 2 : Tests d'Intégration (Adapters)

**Fichier de tests : `tests/integration/test_in_memory_repository.py`**


```python
import pytest
from decimal import Decimal
from product_catalog.domain.entities.product import Product
from product_catalog.infrastructure.adapters.in_memory_product_repository import InMemoryProductRepository

@pytest.fixture
def in_memory_repo():
    return InMemoryProductRepository()

def test_in_memory_repository_save_and_find_by_id(in_memory_repo):
    product = Product("Laptop", "Puissant", Decimal("1200.00"), 10)
    saved_product = in_memory_repo.save(product)
    
    found_product = in_memory_repo.find_by_id(saved_product.id)
    assert found_product == saved_product
    assert found_product.name == "Laptop"

def test_in_memory_repository_find_all(in_memory_repo):
    product1 = Product("Laptop", "Puissant", Decimal("1200.00"), 10)
    product2 = Product("Mouse", "Ergonomic", Decimal("25.00"), 50)
    in_memory_repo.save(product1)
    in_memory_repo.save(product2)
    
    all_products = in_memory_repo.find_all()
    assert len(all_products) == 2
    assert product1 in all_products
    assert product2 in all_products

def test_in_memory_repository_update(in_memory_repo):
    product = Product("Laptop", "Puissant", Decimal("1200.00"), 10)
    saved_product = in_memory_repo.save(product)
    
    saved_product.update_name("Super Laptop")
    saved_product.change_price(Decimal("1300.00"))
    updated_product = in_memory_repo.update(saved_product)
    
    found_product = in_memory_repo.find_by_id(updated_product.id)
    assert found_product.name == "Super Laptop"
    assert found_product.price == Decimal("1300.00")

def test_in_memory_repository_update_non_existent(in_memory_repo):
    non_existent_product = Product("Non Existent", "Desc", Decimal("10"), 1, id="fake_id")
    with pytest.raises(ValueError, match="Produit avec l'ID fake_id non trouvé pour la mise à jour."):
        in_memory_repo.update(non_existent_product)

def test_in_memory_repository_delete(in_memory_repo):
    product = Product("Laptop", "Puissant", Decimal("1200.00"), 10)
    saved_product = in_memory_repo.save(product)
    
    in_memory_repo.delete(saved_product.id)
    
    found_product = in_memory_repo.find_by_id(saved_product.id)
    assert found_product is None

def test_in_memory_repository_delete_non_existent(in_memory_repo):
    with pytest.raises(ValueError, match="Produit avec l'ID fake_id non trouvé pour la suppression."):
        in_memory_repo.delete("fake_id")

def test_in_memory_repository_find_by_id_non_existent(in_memory_repo):
    found_product = in_memory_repo.find_by_id("non_existent_id")
    assert found_product is None
```


#### Partie 3 : Discussion sur les Stratégies de Déploiement

**Impact de la Clean Architecture sur le Déploiement :**
La Clean Architecture, avec sa séparation stricte des préoccupations et son principe d'inversion des dépendances, offre des avantages significatifs pour le déploiement. Le fait que le domaine (Entités, Use Cases) soit indépendant de l'infrastructure signifie que les décisions de déploiement peuvent être prises plus tard dans le cycle de vie du projet et modifiées sans impacter le cœur métier. Cela facilite le "plug-and-play" de différentes technologies d'infrastructure (bases de données, frameworks web, services externes) sans toucher à la logique métier. Le domaine étant agnostique, il peut être déployé dans n'importe quel environnement compatible avec le langage de programmation choisi.

**Options de Déploiement :**

1.  **Monolithe :**
    *   **Description :** L'application entière (domaine, application, infrastructure) est packagée et déployée comme une seule unité. Pour notre catalogue de produits, cela signifierait une seule application web (ex: Flask/Django/Spring Boot) intégrant tous les Use Cases et utilisant un adaptateur de persistance unique.
    *   **Avantages :** Simplicité de développement et de déploiement initial, gestion unique de la configuration.
    *   **Inconvénients :** Scalabilité limitée (tout doit scaler ensemble), rigidité des technologies, impact des changements sur l'ensemble de l'application.
    *   **Pour le catalogue :** Adapté pour un petit catalogue avec un trafic modéré.

2.  **Microservices :**
    *   **Description :** L'application est décomposée en services plus petits, indépendants et déployables séparément, chacun gérant un domaine métier spécifique. Avec la Clean Architecture, chaque microservice pourrait avoir sa propre structure Clean Architecture. Par exemple, un service `ProductCatalogService` gérant les produits, un `InventoryService` gérant les stocks, etc.
    *   **Avantages :** Scalabilité indépendante des services, résilience accrue, flexibilité technologique (chaque service peut utiliser sa propre stack), équipes indépendantes.
    *   **Inconvénients :** Complexité de gestion accrue (réseau, découverte de services, transactions distribuées), latence potentielle.
    *   **Pour le catalogue :** Idéal pour un grand catalogue, avec des exigences de haute disponibilité et de scalabilité, où la gestion des produits et des stocks pourrait être séparée.

3.  **Serverless (Fonctions as a Service - FaaS) :**
    *   **Description :** Chaque Use Case (ou un petit groupe de Use Cases) est déployé comme une fonction sans serveur (ex: AWS Lambda, Google Cloud Functions). L'infrastructure est entièrement gérée par le fournisseur cloud.
    *   **Avantages :** Coût basé sur l'utilisation réelle, scalabilité automatique et quasi-infinie, aucune gestion de serveurs.
    *   **Inconvénients :** "Cold starts" (latence au démarrage), limitations de durée d'exécution et de mémoire, complexité de gestion des états persistants, dépendance au fournisseur cloud.
    *   **Pour le catalogue :** Peut être envisagé pour des opérations spécifiques et peu fréquentes (ex: un Use Case de "génération de rapport de stock"), mais moins adapté pour un service CRUD interactif à haute fréquence sans une architecture bien pensée autour des bases de données serverless.

**Rôle des Conteneurs et de l'Orchestration :**
*   **Conteneurs (Docker) :** Les conteneurs encapsulent l'application et toutes ses dépendances dans une unité légère et portable. Pour une application Clean Architecture, cela signifie que le code du domaine, de l'application et de l'infrastructure est packagé ensemble. Les conteneurs garantissent que l'application s'exécutera de manière identique de l'environnement de développement à la production, éliminant les problèmes de "ça marche sur ma machine". Ils sont essentiels pour la cohérence des déploiements, qu'ils soient monolithiques ou microservices.
*   **Orchestration (Kubernetes, Docker Swarm) :** Les orchestrateurs gèrent le déploiement, la mise à l'échelle, la haute disponibilité et la gestion du cycle de vie des conteneurs. Pour une application Clean Architecture, cela permet de :
    *   **Déploiement automatisé :** Déployer de nouvelles versions de l'application ou de services spécifiques.
    *   **Scalabilité :** Augmenter ou diminuer le nombre d'instances de l'application ou de services en fonction de la charge.
    *   **Haute disponibilité :** Redémarrer automatiquement les conteneurs défaillants et distribuer la charge.
    *   **Gestion des ressources :** Allouer efficacement les ressources CPU et mémoire.
    La Clean Architecture, en favorisant des composants bien isolés, rend l'application plus "container-friendly" et donc plus facile à orchestrer.

**Intégration Continue / Déploiement Continu (CI/CD) :**
Une chaîne CI/CD est cruciale pour automatiser le processus de livraison logicielle. Pour une application Clean Architecture, elle inclurait les étapes clés suivantes :
1.  **Build :** Compilation du code (si nécessaire) et création des artefacts (ex: image Docker).
2.  **Tests Unitaires :** Exécution automatique de tous les tests unitaires (Partie 1) pour valider le cœur métier et les Use Cases. Ces tests sont rapides et fournissent un feedback immédiat.
3.  **Tests d'Intégration :** Exécution des tests d'intégration (Partie 2) pour vérifier l'interaction entre les adaptateurs et les systèmes externes (bases de données, services tiers). Cela peut impliquer le démarrage de services dépendants via Testcontainers ou Docker Compose.
4.  **Tests End-to-End (E2E) :** (Non couvert par ce TP, mais important) Vérification du fonctionnement de l'application complète du point de vue de l'utilisateur.
5.  **Analyse de Qualité :** Outils d'analyse statique de code (SonarQube, pylint) pour détecter les "code smells" et les vulnérabilités.
6.  **Déploiement :** Déploiement automatisé de l'application dans un environnement de staging, puis de production, souvent via des stratégies comme le "blue/green deployment" ou les "canary releases" pour minimiser les risques.
La Clean Architecture facilite cette chaîne en rendant les tests unitaires rapides et fiables, et en isolant les dépendances d'infrastructure pour les tests d'intégration.

---

### Chapitre 2 : Tests Avancés et Déploiement Orienté Microservices/Événements

Cette solution explore des aspects plus avancés des tests, notamment la gestion des erreurs dans les Use Cases, et approfondit les stratégies de déploiement pour des architectures plus distribuées, en s'appuyant sur les avantages de la Clean Architecture.

#### Code de l'Application

Nous allons enrichir l'application avec des exceptions personnalisées pour une gestion d'erreurs plus explicite dans le domaine et les Use Cases.

**Structure du projet :**


```
product_catalog_advanced/
├── domain/
│   ├── entities/
│   │   └── product.py
│   ├── exceptions/
│   │   └── product_exceptions.py
│   └── ports/
│       ├── product_repository.py
│       └── product_output_port.py
├── application/
│   └── use_cases/
│       ├── create_product_use_case.py
│       └── ... (autres use cases similaires)
└── infrastructure/
    └── adapters/
        └── in_memory_product_repository.py
```


**`product_catalog_advanced/domain/exceptions/product_exceptions.py`**


```python
class ProductError(Exception):
    """Classe de base pour les exceptions liées aux produits."""
    pass

class ProductNotFoundError(ProductError):
    """Exception levée quand un produit n'est pas trouvé."""
    pass

class InvalidProductDataError(ProductError):
    """Exception levée quand les données d'un produit sont invalides."""
    pass
```


**`product_catalog_advanced/domain/entities/product.py`** (modifié pour utiliser les exceptions)


```python
import uuid
from decimal import Decimal
from product_catalog_advanced.domain.exceptions.product_exceptions import InvalidProductDataError

class Product:
    def __init__(self, name: str, description: str, price: Decimal, stock: int, id: str = None):
        if not name:
            raise InvalidProductDataError("Le nom du produit ne peut pas être vide.")
        if price <= 0:
            raise InvalidProductDataError("Le prix du produit doit être strictement positif.")
        if stock < 0:
            raise InvalidProductDataError("Le stock du produit ne peut pas être négatif.")

        self.id = id if id else str(uuid.uuid4())
        self.name = name
        self.description = description
        self.price = price
        self.stock = stock

    def update_name(self, new_name: str):
        if not new_name:
            raise InvalidProductDataError("Le nom du produit ne peut pas être vide.")
        self.name = new_name

    def change_price(self, new_price: Decimal):
        if new_price <= 0:
            raise InvalidProductDataError("Le prix du produit doit être strictement positif.")
        self.price = new_price

    def adjust_stock(self, quantity: int):
        if self.stock + quantity < 0:
            raise InvalidProductDataError("Le stock ne peut pas devenir négatif.")
        self.stock += quantity

    def __eq__(self, other):
        if not isinstance(other, Product):
            return NotImplemented
        return self.id == other.id

    def __hash__(self):
        return hash(self.id)
```


**`product_catalog_advanced/application/use_cases/create_product_use_case.py`** (modifié pour gérer les exceptions)


```python
from decimal import Decimal
from product_catalog_advanced.domain.entities.product import Product
from product_catalog_advanced.domain.ports.product_repository import ProductRepository
from product_catalog_advanced.domain.ports.product_output_port import ProductOutputPort
from product_catalog_advanced.domain.exceptions.product_exceptions import InvalidProductDataError

class CreateProductUseCase:
    def __init__(self, repository: ProductRepository, presenter: ProductOutputPort):
        self.repository = repository
        self.presenter = presenter

    def execute(self, name: str, description: str, price: Decimal, stock: int):
        try:
            product = Product(name, description, price, stock) # Lève InvalidProductDataError
            self.repository.save(product)
            self.presenter.present_product(product)
        except InvalidProductDataError as e:
            self.presenter.present_error(str(e))
            raise # Re-lancer pour que le contrôleur puisse aussi gérer si nécessaire
```


*(Les autres Use Cases seraient également modifiés pour lever et/ou attraper `ProductNotFoundError` ou `InvalidProductDataError`.)*

#### Partie 1 : Tests Unitaires (Core Domain) - Focus sur les Exceptions

**Fichier de tests : `tests/unit/test_domain_advanced.py`**


```python
import pytest
from decimal import Decimal
from product_catalog_advanced.domain.entities.product import Product
from product_catalog_advanced.domain.exceptions.product_exceptions import InvalidProductDataError, ProductNotFoundError
from unittest.mock import Mock

# 1.1. Tests des Entités (Product) avec exceptions
def test_product_creation_invalid_name_raises_exception():
    with pytest.raises(InvalidProductDataError, match="Le nom du produit ne peut pas être vide."):
        Product("", "Description", Decimal("10.00"), 5)

def test_product_adjust_stock_negative_result_raises_exception():
    product = Product("Test", "Desc", Decimal("100"), 5)
    with pytest.raises(InvalidProductDataError, match="Le stock ne peut pas devenir négatif."):
        product.adjust_stock(-6)

# 1.2. Tests des Cas d'Utilisation (Use Cases) avec gestion d'exceptions
@pytest.fixture
def mock_repository_advanced():
    return Mock(spec=ProductRepository)

@pytest.fixture
def mock_presenter_advanced():
    return Mock(spec=ProductOutputPort)

from product_catalog_advanced.application.use_cases.create_product_use_case import CreateProductUseCase

def test_create_product_invalid_data_handles_exception(mock_repository_advanced, mock_presenter_advanced):
    use_case = CreateProductUseCase(mock_repository_advanced, mock_presenter_advanced)
    name, desc, price, stock = "", "Puissant", Decimal("1200.00"), 10
    
    with pytest.raises(InvalidProductDataError): # Le Use Case re-lève l'exception
        use_case.execute(name, desc, price, stock)
    
    mock_repository_advanced.save.assert_not_called()
    mock_presenter_advanced.present_error.assert_called_once_with("Le nom du produit ne peut pas être vide.")
    mock_presenter_advanced.present_product.assert_not_called()

# Exemple pour GetProductByIdUseCase (nécessiterait une modification du Use Case pour lever ProductNotFoundError)
# from product_catalog_advanced.application.use_cases.get_product_by_id_use_case import GetProductByIdUseCase
# def test_get_product_by_id_not_found_raises_exception(mock_repository_advanced, mock_presenter_advanced):
#     use_case = GetProductByIdUseCase(mock_repository_advanced, mock_presenter_advanced)
#     product_id = "non_existent_id"
#     mock_repository_advanced.find_by_id.return_value = None
#     
#     with pytest.raises(ProductNotFoundError):
#         use_case.execute(product_id)
#     
#     mock_repository_advanced.find_by_id.assert_called_once_with(product_id)
#     mock_presenter_advanced.present_error.assert_called_once_with(f"Produit avec l'ID {product_id} non trouvé.")
#     mock_presenter_advanced.present_product.assert_not_called()
```


#### Partie 2 : Tests d'Intégration (Adapters) - Avec Base de Données Réelle (Concept)

Pour cette partie, nous discuterons de l'approche plutôt que de fournir un code complet, car cela dépend fortement de la base de données et de l'ORM choisis.

**2.1. Tests de l'Adaptateur `InMemoryProductRepository` :** Les tests seraient identiques à ceux du Chapitre 1, car l'implémentation en mémoire ne change pas fondamentalement avec l'introduction d'exceptions (elle les lève déjà ou les gère).

**2.2. Tests avec une base de données réelle (ex: PostgreSQL avec SQLAlchemy) :**
*   **Configuration :** Utiliser `pytest-docker` ou `Testcontainers` pour démarrer un conteneur PostgreSQL propre pour chaque exécution de test ou suite de tests.
*   **Nettoyage :** Chaque test ou fixture de test doit s'assurer que la base de données est dans un état connu (vide ou avec des données de base) avant son exécution.
*   **Implémentation de l'adaptateur :** Créer une classe `SQLAlchemyProductRepository` qui implémente `ProductRepository` et utilise SQLAlchemy pour interagir avec PostgreSQL.
*   **Tests :** Les tests d'intégration pour cet adaptateur seraient similaires aux tests de l'`InMemoryProductRepository`, mais ils vérifieraient que les données sont correctement persistées et récupérées de la base de données réelle.
    *   Exemple :
        1.  Instancier `SQLAlchemyProductRepository` avec une session de base de données.
        2.  Créer un `Product` et appeler `repository.save(product)`.
        3.  Interroger directement la base de données (ou via une nouvelle instance du repository) pour vérifier que le produit est bien là.
        4.  Tester les cas d'erreur (ex: `ProductNotFoundError` si un ID inexistant est recherché).

#### Partie 3 : Discussion sur les Stratégies de Déploiement Avancées

**Impact de la Clean Architecture sur le Déploiement :**
La Clean Architecture est un atout majeur pour les déploiements complexes. Son indépendance vis-à-vis de l'infrastructure permet une grande flexibilité dans le choix des technologies et des stratégies de déploiement. Le domaine métier, stable et testé, peut être réutilisé dans différents contextes de déploiement (monolithe, microservice, serverless) sans modification. Cela réduit le risque de "vendor lock-in" et facilite les migrations technologiques.

**Options de Déploiement (approfondissement) :**

1.  **Microservices avec Décomposition par Bounded Context :**
    *   **Description :** La Clean Architecture s'aligne naturellement avec la décomposition en microservices basée sur le Domain-Driven Design (DDD) et les "Bounded Contexts". Chaque Bounded Context (ex: "Catalogue de Produits", "Gestion des Stocks", "Commandes") peut être implémenté comme un microservice distinct, chacun ayant sa propre Clean Architecture interne.
    *   **Avantages :** Chaque service est autonome, avec son propre cycle de vie de déploiement et sa propre stack technologique. Cela permet une scalabilité horizontale fine et une résilience accrue.
    *   **Inconvénients :** Complexité opérationnelle (gestion du réseau, de la découverte, de la traçabilité, des transactions distribuées). Nécessite des mécanismes de communication inter-services robustes (REST, gRPC, Message Brokers).
    *   **Pour le catalogue :** Le `ProductCatalogService` pourrait être un microservice, le `InventoryService` un autre, le `OrderService` un troisième. Ils communiqueraient via des APIs ou un bus d'événements.

2.  **Architecture Événementielle (Event-Driven Architecture - EDA) :**
    *   **Description :** Les services communiquent principalement via des événements asynchrones. Un service publie un événement (ex: `ProductCreatedEvent`) et d'autres services y souscrivent pour réagir. Cela découple fortement les services et permet une grande flexibilité.
    *   **Avantages :** Découplage temporel et spatial, résilience (les services peuvent tomber et se remettre en ligne sans bloquer la chaîne), scalabilité (les consommateurs peuvent être scalés indépendamment), auditabilité (le journal des événements est un historique complet).
    *   **Inconvénients :** Complexité de conception (gestion de l'idempotence, de l'ordre des événements, des transactions distribuées), difficulté de débogage des flux asynchrones.
    *   **Pour le catalogue :** Lorsqu'un produit est créé (`ProductCreatedEvent`), un service d'indexation de recherche pourrait écouter cet événement pour mettre à jour son index, un service de notification pourrait alerter les administrateurs, etc.

**Rôle des Conteneurs et de l'Orchestration (approfondissement) :**
*   **Conteneurs :** Dans une architecture microservices, chaque microservice est généralement conteneurisé. La Clean Architecture garantit que le code métier de chaque microservice est bien isolé, ce qui facilite sa conteneurisation. Les adaptateurs d'infrastructure spécifiques à chaque microservice sont inclus dans son conteneur.
*   **Orchestration (Kubernetes) :** Kubernetes devient l'épine dorsale d'un déploiement microservices. Il gère :
    *   **Déploiement :** Déploiement de chaque microservice indépendamment.
    *   **Mise à l'échelle automatique :** Scaler les services en fonction de la charge.
    *   **Découverte de services :** Les services peuvent se trouver et communiquer entre eux.
    *   **Gestion de la configuration :** Gérer les secrets et les configurations spécifiques à chaque environnement.
    *   **Résilience :** Redémarrage automatique, réplication, auto-réparation.
    La Clean Architecture, en produisant des composants métier testables et isolés, permet de construire des microservices robustes qui s'intègrent bien dans un écosystème Kubernetes.

**Intégration Continue / Déploiement Continu (CI/CD) pour Architectures Distribuées :**
Une chaîne CI/CD pour une architecture Clean Architecture distribuée est plus complexe mais aussi plus puissante :
1.  **Mono-repo vs Multi-repo :** Décision d'avoir tous les services dans un seul dépôt Git (mono-repo) ou chaque service dans son propre dépôt (multi-repo). La Clean Architecture est compatible avec les deux.
2.  **Pipelines par Service :** Chaque microservice (ou Bounded Context) aurait son propre pipeline CI/CD, permettant des déploiements indépendants.
3.  **Tests :**
    *   **Unitaires :** Exécutés rapidement pour chaque service.
    *   **Intégration :** Tests des adaptateurs de chaque service.
    *   **Contract Tests :** Vérification que les APIs des services respectent les contrats attendus par leurs consommateurs (ex: Pact, Spring Cloud Contract). Cela est crucial dans une architecture distribuée pour éviter les régressions.
    *   **End-to-End :** Tests de flux utilisateur complets impliquant plusieurs services.
4.  **Déploiement :** Utilisation de stratégies avancées (Canary, Blue/Green) pour déployer les services avec un risque minimal. Les outils d'orchestration comme Kubernetes facilitent ces stratégies.
5.  **Observabilité :** Intégration de la télémétrie (logs, métriques, traces) à chaque étape du pipeline et dans l'application déployée pour surveiller la santé et les performances des services.