
## TP : Démêler le Monolithe - Analyse et Premiers Pas vers la Clean Architecture

### Contexte

Comprendre les défis inhérents aux architectures monolithiques est une étape fondamentale pour quiconque aspire à concevoir des systèmes robustes et maintenables. Avant de pouvoir construire une architecture propre, il est essentiel de savoir identifier les signaux d'alerte dans une base de code existante. Ce TP vous mettra dans la peau d'un architecte logiciel chargé d'évaluer une application monolithique et de proposer des améliorations.

### Objectif Pédagogique

L'objectif de ce TP est de vous permettre d'analyser une architecture monolithique existante, d'identifier les dépendances fortes, les couplages excessifs et les violations potentielles des principes de la Clean Architecture, puis de proposer des pistes concrètes d'amélioration.

### Scénario du Mini-Projet : L'application "ProductCatalog"

Vous êtes confronté à une application web existante, nommée "ProductCatalog", développée il y a quelques années. Elle gère un catalogue de produits, des catégories associées. L'application est un monolithe classique, structurée de manière courante pour ce type d'architecture :

*   **`Controllers` (ou `Routes`)** : Gèrent les requêtes HTTP, appellent la logique métier et préparent les données pour la vue.
*   **`Services` (ou `BusinessLogic`)** : Contiennent la logique métier principale. Ils interagissent souvent directement avec les `Repositories`.
*   **`Repositories` (ou `DataAccess`)** : Gèrent l'accès aux données (base de données, fichiers, etc.).
*   **`Models` (ou `Entities`)** : Représentent les structures de données (Produit, Catégorie) et sont souvent utilisées à travers toutes les couches.

**Exemples de problématiques courantes que vous pourriez rencontrer dans "ProductCatalog" :**

*   **Couplage fort entre couches :** Un `Controller` peut instancier directement un `Repository` ou un `Service` sans passer par des interfaces ou l'injection de dépendances.
*   **Logique métier dispersée :** Des règles métier importantes peuvent être implémentées directement dans les `Controllers`, dans les `Services`, voire même dans les `Repositories`.
*   **Entités anémiques et omniprésentes :** Les objets `Product`, `Category` sont passés et manipulés directement par toutes les couches (Web, Service, Repository), agissant à la fois comme DTOs, objets de domaine et objets de persistance.
*   **Dépendances inversées non respectées :** Les couches de haut niveau (logique métier) dépendent des couches de bas niveau (accès aux données, détails d'implémentation).
*   **Mélange des préoccupations (Separation of Concerns) :** Un `Service` peut contenir de la logique de validation, de la logique de persistance, et même des éléments de formatage pour la présentation.

### Travail Demandé

Votre mission consiste à analyser cette application "ProductCatalog" et à documenter vos observations.

1.  **Prise de Connaissance du Code :**
    *   Le code est disponible au bas de ce document.
    *   Si nécessaire, vous utilisez une IA générative (ChatGPT, Copilot, etc.) pour traduire dans un langage et un framework que vous maîtrisez (ex: C#/.NET, Java/Spring, Python/Flask/Django, Node.js/Express).
    *   Assurez-vous que le code généré illustre *clairement* les problématiques monolithiques mentionnées (dépendances fortes, mélange des préoccupations, entités omniprésentes).
    *   Prenez le temps de comprendre la structure et le fonctionnement de ce code.

2.  **Identification des Dépendances et Couplages Forts :**
    *   Parcourez le code.
    *   Identifiez et listez les dépendances directes et les instanciations concrètes entre les différentes couches et modules (ex: `ProductController` instanciant `ProductService`, `ProductService` instanciant `ProductRepository`, etc.).
    *   Mettez en évidence les endroits où une modification dans une partie du code pourrait avoir un impact significatif et inattendu sur d'autres parties.

3.  **Analyse des Violations des Principes de Clean Architecture :**
    *   En vous basant sur votre analyse des dépendances, identifiez les violations spécifiques des principes de la Clean Architecture (ou des principes SOLID qui en sont la base).
    *   Pour chaque violation, expliquez en quoi elle pose problème (ex: "Tel service dépend directement de ... , violant le principe d'inversion de dépendance, ce qui rend les tests unitaires difficiles et le changement de persistance complexe.").
    *   Concentrez-vous sur les concepts clés :
        *   **Indépendance du Framework :** Le code métier est-il lié à un framework web ou de persistance ?
        *   **Indépendance de l'UI :** La logique métier est-elle affectée par des changements d'interface utilisateur ?
        *   **Indépendance de la Base de Données :** La logique métier est-elle directement liée à la base de données ?
        *   **Testabilité :** Les composants métier sont-ils facilement testables isolément ?
        *   **Règles Métier :** Où se trouvent les règles métier ? Sont-elles claires et isolées ?
        *   **Principe d'Inversion de Dépendance (DIP) :** Les modules de haut niveau dépendent-ils des abstractions, et non des implémentations ?

4.  **Proposition de Pistes d'Amélioration :**
    *   Pour chaque problématique identifiée, proposez des pistes d'amélioration concrètes en vous appuyant sur les principes de la Clean Architecture.
    *   Décrivez comment vous pourriez restructurer le code pour réduire le couplage et améliorer la séparation des préoccupations.
    *   N'hésitez pas à esquisser des changements de structure de dossiers, l'introduction d'interfaces, de DTOs (Data Transfer Objects), de Use Cases (ou Interactors), ou l'utilisation de l'injection de dépendances.
    *   Vous n'avez pas à réécrire l'application, mais à présenter une vision claire des étapes initiales de refactoring.

### Ressources et Outils

*   **Votre environnement de développement habituel.**
*   **Documentation sur la Clean Architecture :** Le cours, Le livre de Robert C. Martin ("Clean Architecture") ou des articles de référence en ligne.

### Livrables

Un rapport structuré (document texte, Markdown, ou PDF) contenant les sections suivantes :

1.  **Introduction :** Brève description de l'application "ProductCatalog" générée.
2.  **Analyse des Dépendances et Couplages :** Liste des points de couplage identifiés, avec des extraits de code pertinents pour illustrer.
3.  **Violations des Principes de Clean Architecture :** Pour chaque violation, une explication claire et un lien avec le code.
4.  **Pistes d'Amélioration :** Propositions concrètes pour chaque problématique, expliquant comment elles s'alignent avec la Clean Architecture. Des diagrammes simples ou des pseudo-codes peuvent être inclus.
5.  **Conclusion :** Vos réflexions sur l'exercice et les principaux apprentissages.

### Conseils pour les Apprenants

*   **Soyez méthodique :** Abordez l'analyse couche par couche ou fonctionnalité par fonctionnalité.
*   **Ne cherchez pas la perfection immédiate :** L'objectif est d'identifier les problèmes et de proposer des directions, pas de livrer une architecture parfaite du premier coup.
*   **Si besoin, utilisez l'IA intelligemment :** L'IA est un outil puissant pour la génération de code et l'analyse. N'hésitez pas à lui demander des explications sur des patterns, ou des suggestions de refactoring. Cependant, gardez un esprit critique et validez toujours ses propositions. C'est votre compréhension qui est évaluée.
*   **Documentez vos choix :** Justifiez pourquoi vous considérez une partie du code comme un couplage fort ou une violation d'un principe.

---

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
from models import Product

class ProductService:
    def __init__(self, db_session):
        self.product_repo = ProductRepository(db_session)

    def get_products_for_display(self):
        products = self.product_repo.get_all()
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

# Initialisation de la base de données
engine = create_engine(config.DATABASE_URL)
Base.metadata.create_all(engine)
Session = sessionmaker(bind=engine)

@app.route('/products', methods=['GET'])
def list_products():
    db_session = Session()
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
