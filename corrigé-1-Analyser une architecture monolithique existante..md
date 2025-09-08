Voici deux approches pour aborder le TP "Démêler le Monolithe - Analyse et Premiers Pas vers la Clean Architecture", présentées sous forme de chapitres distincts.

---

### Chapitre 1 : Approche Incrémentale - Maîtrise du DIP et Introduction des Abstractions

Cette première solution se concentre sur une approche pragmatique et incrémentale. L'objectif est d'identifier les points de douleur les plus immédiats dans le monolithe et de les adresser en introduisant des abstractions clés, notamment via le Principe d'Inversion de Dépendance (DIP), pour améliorer la testabilité et la flexibilité.

#### 1. Génération et Prise de Connaissance du Code (Simulé)

**Structure du code monolithique simulé :**


```
product_catalog/
├── app.py                  # Points d'entrée HTTP (routes), logique de contrôleur
├── services.py             # Logique métier, interagit directement avec les repositories
├── repositories.py         # Accès direct à la base de données (ex: SQLAlchemy)
├── models.py               # Modèles de données (ORM), utilisés partout
└── config.py               # Configuration de l'application
```


**Extrait de code représentatif des problématiques :**


```python
# models.py
from sqlalchemy import create_engine, Column, Integer, String, Float, ForeignKey
from sqlalchemy.orm import sessionmaker, relationship
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Product(Base):
    __tablename__ = 'products'
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    description = Column(String)
    price = Column(Float, nullable=False)
    category_id = Column(Integer, ForeignKey('categories.id'))
    category = relationship("Category", back_populates="products")

class Category(Base):
    __tablename__ = 'categories'
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    products = relationship("Product", order_by=Product.id, back_populates="category")

# repositories.py
from models import Product, Category, Base, sessionmaker, create_engine

class ProductRepository:
    def __init__(self, db_session):
        self.db_session = db_session

    def get_all(self):
        return self.db_session.query(Product).all()

    def get_by_id(self, product_id):
        return self.db_session.query(Product).filter_by(id=product_id).first()

    def add(self, product):
        self.db_session.add(product)
        self.db_session.commit()
        return product

# services.py
from repositories import ProductRepository
from models import Product # Utilisation directe du modèle ORM

class ProductService:
    def __init__(self, db_session):
        # Couplage fort: instancie directement le repository concret
        self.product_repo = ProductRepository(db_session)

    def get_products_for_display(self):
        products = self.product_repo.get_all()
        # Logique métier mélangée avec la préparation des données pour l'UI
        return [{"id": p.id, "name": p.name, "price": f"{p.price:.2f} EUR"} for p in products]

    def create_product(self, name, description, price, category_id):
        # Validation simple ici, mais pourrait être plus complexe et dispersée
        if price <= 0:
            raise ValueError("Price must be positive.")
        new_product = Product(name=name, description=description, price=price, category_id=category_id)
        return self.product_repo.add(new_product)

# app.py (Flask routes)
from flask import Flask, jsonify, request
from services import ProductService
from models import Base, create_engine, sessionmaker
import config

app = Flask(__name__)

# Initialisation de la base de données (souvent globale ou passée partout)
engine = create_engine(config.DATABASE_URL)
Base.metadata.create_all(engine)
Session = sessionmaker(bind=engine)

@app.route('/products', methods=['GET'])
def list_products():
    db_session = Session()
    # Couplage fort: le contrôleur instancie directement le service concret
    product_service = ProductService(db_session)
    products_data = product_service.get_products_for_display()
    db_session.close()
    return jsonify(products_data)

@app.route('/products', methods=['POST'])
def add_product():
    db_session = Session()
    product_service = ProductService(db_session)
    data = request.get_json()
    try:
        new_product = product_service.create_product(
            data['name'], data.get('description'), data['price'], data['category_id']
        )
        db_session.close()
        return jsonify({"message": "Product created", "id": new_product.id}), 201
    except ValueError as e:
        db_session.rollback()
        db_session.close()
        return jsonify({"error": str(e)}), 400
    except Exception as e:
        db_session.rollback()
        db_session.close()
        return jsonify({"error": "An unexpected error occurred"}), 500

if __name__ == '__main__':
    app.run(debug=True)
```


#### 2. Identification des Dépendances et Couplages Forts

*   **`app.py` (Controller) dépend directement de `services.py` (Service Concret)** : `product_service = ProductService(db_session)`. Le contrôleur est lié à une implémentation spécifique du service.
*   **`services.py` (Service) dépend directement de `repositories.py` (Repository Concret)** : `self.product_repo = ProductRepository(db_session)`. Le service est lié à une implémentation spécifique de l'accès aux données.
*   **`services.py` utilise directement les `models.py` (ORM Entities)** : `new_product = Product(...)`. Les entités de persistance sont exposées et manipulées directement par la logique métier, mélangeant les préoccupations de domaine et de persistance.
*   **Logique métier dispersée** : La validation du prix est dans `ProductService`, mais d'autres validations pourraient être dans le contrôleur ou même le modèle. La préparation des données pour l'affichage (`get_products_for_display`) est également dans le service, ce qui n'est pas sa responsabilité principale.
*   **Initialisation de la base de données** : L'initialisation de l'engine SQLAlchemy et de la session est directement dans `app.py`, couplant l'application web aux détails de la base de données.

#### 3. Analyse des Violations des Principes de Clean Architecture

*   **Principe d'Inversion de Dépendance (DIP) violé** : Les modules de haut niveau (`ProductService`, `app.py`) dépendent des modules de bas niveau (`ProductRepository`, `ProductService` concret). Cela rend difficile le remplacement des implémentations (ex: changer de base de données ou de framework ORM) et la testabilité unitaire.
*   **Indépendance de la Base de Données violée** : La logique métier dans `ProductService` manipule directement les objets `Product` de SQLAlchemy, liant le domaine à l'ORM et à la base de données.
*   **Indépendance du Framework / de l'UI violée** : `ProductService.get_products_for_display()` prépare les données pour l'UI, ce qui signifie que des changements dans l'UI pourraient impacter la logique métier. Le contrôleur instancie directement le service, le liant au framework web.
*   **Mélange des préoccupations (Separation of Concerns)** :
    *   Le `ProductService` gère à la fois la logique métier, la validation et la transformation des données pour l'affichage.
    *   Les `Models` (entités ORM) servent à la fois de DTOs, et d'objets de persistance.
*   **Testabilité faible** : Tester `ProductService` nécessite une session de base de données réelle ou un mock complexe de `ProductRepository`, car il dépend de son implémentation concrète.

#### 4. Proposition de Pistes d'Amélioration

L'objectif est d'introduire des interfaces (ou classes abstraites en Python) pour inverser les dépendances et de mieux séparer les préoccupations.

1.  **Introduction d'Interfaces pour les Repositories :**
    *   Créer une interface `IProductRepository` (ou `AbstractProductRepository`) qui définit les méthodes d'accès aux données (`get_all`, `get_by_id`, `add`).
    *   `ProductRepository` implémentera cette interface.
    *   Le `ProductService` dépendra de l'interface, et non de l'implémentation concrète.

    
```python
    # interfaces.py
    from abc import ABC, abstractmethod

    class IProductRepository(ABC):
        @abstractmethod
        def get_all(self):
            pass

        @abstractmethod
        def get_by_id(self, product_id):
            pass

        @abstractmethod
        def add(self, product_data): # Prendra des DTOs ou des entités de domaine
            pass

    # repositories.py (implémentation)
    from interfaces import IProductRepository
    from models import Product # Toujours utilisé ici, mais pas dans le service

    class SQLProductRepository(IProductRepository): # Renommé pour clarifier l'implémentation
        def __init__(self, db_session):
            self.db_session = db_session

        def get_all(self):
            return self.db_session.query(Product).all()

        def get_by_id(self, product_id):
            return self.db_session.query(Product).filter_by(id=product_id).first()

        def add(self, product_data):
            # Ici, on mapperait product_data (domaine) vers Product (ORM)
            new_product_orm = Product(
                name=product_data.name,
                description=product_data.description,
                price=product_data.price,
                category_id=product_data.category_id
            )
            self.db_session.add(new_product_orm)
            self.db_session.commit()
            return new_product_orm # Retourne l'ORM, à mapper en entité de domaine si nécessaire
    ```


2.  **Injection de Dépendances pour les Services :**
    *   Le `ProductService` recevra son `IProductRepository` via son constructeur.
    *   Le contrôleur (ou un conteneur d'injection de dépendances) sera responsable de fournir l'implémentation concrète.

    
```python
    # services.py (amélioré)
    from interfaces import IProductRepository
    # from domain_entities import Product as DomainProduct # Utiliserait une entité de domaine

    class ProductService:
        def __init__(self, product_repo: IProductRepository):
            self.product_repo = product_repo

        def get_products_for_display(self):
            products = self.product_repo.get_all() # Retourne des entités ORM ou de domaine
            # La transformation pour l'affichage devrait être déplacée vers une couche de présentation
            return [{"id": p.id, "name": p.name, "price": f"{p.price:.2f} EUR"} for p in products]

        def create_product(self, name, description, price, category_id):
            if price <= 0:
                raise ValueError("Price must be positive.")
            # new_product = DomainProduct(name, description, price, category_id) # Utiliserait une entité de domaine
            # return self.product_repo.add(new_product)
            # Pour l'instant, on passe les données brutes, mais l'idéal serait une entité de domaine ou un DTO
            return self.product_repo.add({'name': name, 'description': description, 'price': price, 'category_id': category_id})
    ```


3.  **Refactorisation du Contrôleur et de l'Initialisation :**
    *   Le contrôleur instanciera le service en lui passant l'implémentation concrète du repository. Idéalement, un conteneur DI gérerait cela.
    *   L'initialisation de la base de données devrait être centralisée, par exemple dans une fonction `create_app` ou un module de configuration.

    
```python
    # app.py (amélioré)
    from flask import Flask, jsonify, request
    from services import ProductService
    from repositories import SQLProductRepository # Implémentation concrète
    from models import Base, create_engine, sessionmaker
    import config

    def create_app():
        app = Flask(__name__)
        engine = create_engine(config.DATABASE_URL)
        Base.metadata.create_all(engine)
        Session = sessionmaker(bind=engine)

        @app.before_request
        def before_request():
            # Gérer la session DB pour chaque requête
            request.db_session = Session()

        @app.after_request
        def after_request(response):
            # Fermer la session DB après chaque requête
            if hasattr(request, 'db_session'):
                request.db_session.close()
            return response

        @app.route('/products', methods=['GET'])
        def list_products():
            # Le contrôleur dépend de l'interface via l'injection
            product_repo = SQLProductRepository(request.db_session)
            product_service = ProductService(product_repo)
            products_data = product_service.get_products_for_display()
            return jsonify(products_data)

        @app.route('/products', methods=['POST'])
        def add_product():
            product_repo = SQLProductRepository(request.db_session)
            product_service = ProductService(product_repo)
            data = request.get_json()
            try:
                new_product = product_service.create_product(
                    data['name'], data.get('description'), data['price'], data['category_id']
                )
                return jsonify({"message": "Product created", "id": new_product.id}), 201
            except ValueError as e:
                request.db_session.rollback()
                return jsonify({"error": str(e)}), 400
            except Exception as e:
                request.db_session.rollback()
                return jsonify({"error": "An unexpected error occurred"}), 500
        return app

    if __name__ == '__main__':
        app = create_app()
        app.run(debug=True)
    ```


Ces étapes initiales réduisent le couplage, améliorent la testabilité (on peut mocker `IProductRepository` pour tester `ProductService`) et préparent le terrain pour une séparation plus poussée.

---

### Chapitre 2 : Approche Orientée Clean Architecture - Introduction des Cas d'Utilisation

Cette deuxième solution propose une restructuration plus ambitieuse, alignée sur les principes fondamentaux de la Clean Architecture. Elle vise à isoler la logique métier pure (les "règles d'entreprise") au centre de l'application, la rendant indépendante des détails techniques.

#### 1. Génération et Prise de Connaissance du Code (Simulé)

Nous nous basons sur le même code monolithique Python/Flask décrit précédemment, avec ses problématiques de couplage fort et de mélange des préoccupations.

#### 2. Identification des Dépendances et Couplages Forts

Les mêmes problématiques de dépendances et de couplages forts sont identifiées :
*   `Controller` dépend de `Service` concret.
*   `Service` dépend de `Repository` concret.
*   `Service` et `Repository` manipulent directement les `Models` ORM.
*   Logique métier et de présentation mélangées dans `Service`.
*   Forte dépendance à Flask et SQLAlchemy à travers l'application.

#### 3. Analyse des Violations des Principes de Clean Architecture

Les violations sont identiques à celles identifiées dans la première approche, mais nous allons les adresser de manière plus systématique :
*   Violation du DIP.
*   Manque d'indépendance de la base de données, de l'UI et du framework.
*   Mélange des préoccupations (logique métier, persistance, présentation).
*   Faible testabilité des composants métier.
*   Les "règles d'entreprise" ne sont pas clairement isolées et peuvent être affectées par des changements externes.

#### 4. Proposition de Pistes d'Amélioration

L'objectif est de créer des couches distinctes et de définir des interfaces claires pour chaque interaction, en plaçant les "Use Cases" (Cas d'Utilisation) au cœur de la logique métier.

1.  **Définition des Couches de la Clean Architecture :**

    *   **Domain (Entities)** : Contient les règles d'entreprise les plus générales et les entités de domaine pures (sans dépendance à l'ORM).
    *   **Application (Use Cases / Interactors)** : Contient les règles d'entreprise spécifiques à l'application. Orchestre le flux de données vers et depuis les entités, et utilise les interfaces de passerelles (Repositories).
    *   **Adapters (Controllers, Presenters, Gateways)** : Convertit les données du format externe (HTTP, DB) vers le format interne (Domain/Application) et vice-versa.
        *   `Controllers` : Adaptateurs d'entrée (Web).
        *   `Presenters` : Adaptateurs de sortie (pour l'UI).
        *   `Repositories` (Gateways) : Adaptateurs de persistance.
    *   **Frameworks & Drivers (Web Framework, DB)** : Les détails d'implémentation.

2.  **Introduction des Entités de Domaine Pures :**
    *   Créer des classes `Product` et `Category` dans un dossier `domain/entities` qui sont de simples objets Python, sans aucune dépendance à SQLAlchemy ou à d'autres frameworks.

    
```python
    # domain/entities.py
    from dataclasses import dataclass

    @dataclass
    class Product:
        id: int = None
        name: str = None
        description: str = None
        price: float = None
        category_id: int = None

    @dataclass
    class Category:
        id: int = None
        name: str = None
    ```


3.  **Définition des Interfaces de Repositories (Gateways) :**
    *   Les interfaces sont définies dans la couche `Application` ou `Domain` (selon la granularité), mais implémentées dans la couche `Adapters`.

    
```python
    # application/interfaces/repositories.py
    from abc import ABC, abstractmethod
    from domain.entities import Product, Category

    class IProductRepository(ABC):
        @abstractmethod
        def get_all(self) -> list[Product]:
            pass

        @abstractmethod
        def get_by_id(self, product_id: int) -> Product:
            pass

        @abstractmethod
        def add(self, product: Product) -> Product:
            pass

    class ICategoryRepository(ABC):
        @abstractmethod
        def get_by_id(self, category_id: int) -> Category:
            pass
    ```


4.  **Création des Cas d'Utilisation (Use Cases / Interactors) :**
    *   Chaque cas d'utilisation représente une action métier spécifique. Il dépend des interfaces de repositories et des entités de domaine.

    
```python
    # application/use_cases/product_use_cases.py
    from application.interfaces.repositories import IProductRepository, ICategoryRepository
    from domain.entities import Product

    class GetProductsUseCase:
        def __init__(self, product_repo: IProductRepository):
            self.product_repo = product_repo

        def execute(self) -> list[Product]:
            return self.product_repo.get_all()

    class CreateProductUseCase:
        def __init__(self, product_repo: IProductRepository, category_repo: ICategoryRepository):
            self.product_repo = product_repo
            self.category_repo = category_repo

        def execute(self, name: str, description: str, price: float, category_id: int) -> Product:
            if price <= 0:
                raise ValueError("Price must be positive.")
            if not self.category_repo.get_by_id(category_id):
                raise ValueError(f"Category with ID {category_id} does not exist.")

            new_product = Product(name=name, description=description, price=price, category_id=category_id)
            return self.product_repo.add(new_product)
    ```


5.  **Implémentation des Repositories (Adapters) :**
    *   Ces classes implémentent les interfaces définies et gèrent l'interaction avec la base de données (SQLAlchemy ORM). Elles mappent les entités ORM vers les entités de domaine et vice-versa.

    
```python
    # infrastructure/repositories/sqlalchemy_product_repository.py
    from application.interfaces.repositories import IProductRepository, ICategoryRepository
    from domain.entities import Product, Category
    from infrastructure.database.models import Product as ORMProduct, Category as ORMCategory # Modèles ORM

    class SQLAlchemyProductRepository(IProductRepository):
        def __init__(self, db_session):
            self.db_session = db_session

        def _to_domain_product(self, orm_product: ORMProduct) -> Product:
            return Product(
                id=orm_product.id,
                name=orm_product.name,
                description=orm_product.description,
                price=orm_product.price,
                category_id=orm_product.category_id
            )

        def _to_orm_product(self, domain_product: Product) -> ORMProduct:
            return ORMProduct(
                id=domain_product.id,
                name=domain_product.name,
                description=domain_product.description,
                price=domain_product.price,
                category_id=domain_product.category_id
            )

        def get_all(self) -> list[Product]:
            orm_products = self.db_session.query(ORMProduct).all()
            return [self._to_domain_product(p) for p in orm_products]

        def get_by_id(self, product_id: int) -> Product:
            orm_product = self.db_session.query(ORMProduct).filter_by(id=product_id).first()
            return self._to_domain_product(orm_product) if orm_product else None

        def add(self, product: Product) -> Product:
            orm_product = self._to_orm_product(product)
            self.db_session.add(orm_product)
            self.db_session.commit()
            return self._to_domain_product(orm_product) # Retourne l'entité de domaine avec l'ID généré

    class SQLAlchemyCategoryRepository(ICategoryRepository):
        def __init__(self, db_session):
            self.db_session = db.session

        def _to_domain_category(self, orm_category: ORMCategory) -> Category:
            return Category(id=orm_category.id, name=orm_category.name)

        def get_by_id(self, category_id: int) -> Category:
            orm_category = self.db_session.query(ORMCategory).filter_by(id=category_id).first()
            return self._to_domain_category(orm_category) if orm_category else None
    ```


6.  **Refactorisation des Contrôleurs (Adapters) :**
    *   Les contrôleurs deviennent de simples adaptateurs qui reçoivent les requêtes HTTP, appellent les cas d'utilisation appropriés, et formatent la réponse. Ils ne contiennent aucune logique métier.

    
```python
    # presentation/controllers/product_controller.py
    from flask import Blueprint, jsonify, request
    from application.use_cases.product_use_cases import GetProductsUseCase, CreateProductUseCase
    from infrastructure.repositories.sqlalchemy_product_repository import SQLAlchemyProductRepository, SQLAlchemyCategoryRepository
    from infrastructure.database.models import Base, create_engine, sessionmaker
    import config

    product_bp = Blueprint('products', __name__)

    # Initialisation de la base de données et gestion des sessions
    engine = create_engine(config.DATABASE_URL)
    Base.metadata.create_all(engine)
    Session = sessionmaker(bind=engine)

    @product_bp.before_request
    def before_request():
        request.db_session = Session()

    @product_bp.after_request
    def after_request(response):
        if hasattr(request, 'db_session'):
            request.db_session.close()
        return response

    @product_bp.route('/products', methods=['GET'])
    def list_products():
        product_repo = SQLAlchemyProductRepository(request.db_session)
        get_products_use_case = GetProductsUseCase(product_repo)
        products = get_products_use_case.execute()
        # Utilisation d'un Presenter ou de DTOs de présentation serait l'étape suivante
        return jsonify([{"id": p.id, "name": p.name, "price": f"{p.price:.2f} EUR"} for p in products])

    @product_bp.route('/products', methods=['POST'])
    def add_product():
        product_repo = SQLAlchemyProductRepository(request.db_session)
        category_repo = SQLAlchemyCategoryRepository(request.db_session)
        create_product_use_case = CreateProductUseCase(product_repo, category_repo)
        data = request.get_json()
        try:
            new_product = create_product_use_case.execute(
                data['name'], data.get('description'), data['price'], data['category_id']
            )
            return jsonify({"message": "Product created", "id": new_product.id}), 201
        except ValueError as e:
            request.db_session.rollback()
            return jsonify({"error": str(e)}), 400
        except Exception as e:
            request.db_session.rollback()
            return jsonify({"error": "An unexpected error occurred"}), 500
    ```


7.  **Structure de Dossiers Finale :**

    
```
    product_catalog/
    ├── application/
    │   ├── interfaces/
    │   │   └── repositories.py   # Interfaces pour les gateways
    │   └── use_cases/
    │       └── product_use_cases.py # Logique métier spécifique à l'application
    ├── domain/
    │   └── entities.py           # Entités de domaine pures
    ├── infrastructure/
    │   ├── database/
    │   │   └── models.py         # Modèles ORM (SQLAlchemy)
    │   └── repositories/
    │       └── sqlalchemy_product_repository.py # Implémentations concrètes des repositories
    ├── presentation/
    │   └── controllers/
    │       └── product_controller.py # Contrôleurs Flask
    ├── config.py
    └── app.py                    # Point d'entrée principal, enregistre les blueprints
    ```


Cette approche, bien que plus lourde au début, offre une séparation des préoccupations maximale. Les règles métier sont au centre, indépendantes de la base de données, du framework web ou de l'UI. La testabilité est grandement améliorée, car les cas d'utilisation peuvent être testés unitairement avec des mocks simples pour les interfaces de repositories.