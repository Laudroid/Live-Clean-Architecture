Voici deux solutions pour le TP de construction d'un module "Gestion de Livres" avec la Clean Architecture, chacune illustrant des choix d'implémentation légèrement différents tout en respectant les principes fondamentaux.

---

### Chapitre 1 : Implémentation Standard de la Clean Architecture (Python)

Cette solution propose une implémentation directe des couches et des ports de la Clean Architecture, en utilisant Python. L'accent est mis sur la séparation claire des responsabilités et l'inversion des dépendances via des interfaces (classes abstraites en Python).

#### 1. Structure du Projet


```
src/
├── domain/
│   ├── entities/
│   │   └── book.py
│   └── ports/
│       ├── input/
│       │   ├── add_book_input_port.py
│       │   ├── list_books_input_port.py
│       │   └── get_book_input_port.py
│       └── output/
│           ├── book_repository_port.py
│           └── book_output_port.py
├── application/
│   └── usecases/
│       ├── add_book_use_case.py
│       ├── list_books_use_case.py
│       └── get_book_use_case.py
└── infrastructure/
    ├── adapters/
    │   ├── controllers/
    │   │   └── book_controller.py
    │   ├── presenters/
    │   │   └── book_console_presenter.py
    │   └── repositories/
    │       └── in_memory_book_repository_adapter.py
    └── main/
        └── app.py
```


#### 2. Code Source

**`src/domain/entities/book.py`**

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class Book:
    """Représente l'entité métier Livre."""
    id: Optional[str] = None
    title: str = ""
    author: str = ""
    isbn: str = ""
```


**`src/domain/ports/input/add_book_input_port.py`**

```python
from abc import ABC, abstractmethod
from typing import Dict, Any

class AddBookInputPort(ABC):
    """Port d'entrée pour l'ajout d'un livre."""
    @abstractmethod
    def add_book(self, book_data: Dict[str, Any]) -> None:
        pass
```


**`src/domain/ports/input/list_books_input_port.py`**

```python
from abc import ABC, abstractmethod

class ListBooksInputPort(ABC):
    """Port d'entrée pour la liste des livres."""
    @abstractmethod
    def list_books(self) -> None:
        pass
```


**`src/domain/ports/input/get_book_input_port.py`**

```python
from abc import ABC, abstractmethod

class GetBookInputPort(ABC):
    """Port d'entrée pour la récupération d'un livre par ID."""
    @abstractmethod
    def get_book_by_id(self, book_id: str) -> None:
        pass
```


**`src/domain/ports/output/book_repository_port.py`**

```python
from abc import ABC, abstractmethod
from typing import List, Optional
from src.domain.entities.book import Book

class BookRepositoryPort(ABC):
    """Port de sortie pour la persistance des livres."""
    @abstractmethod
    def save(self, book: Book) -> Book:
        pass

    @abstractmethod
    def find_all(self) -> List[Book]:
        pass

    @abstractmethod
    def find_by_id(self, book_id: str) -> Optional[Book]:
        pass

    @abstractmethod
    def find_by_isbn(self, isbn: str) -> Optional[Book]:
        pass
```


**`src/domain/ports/output/book_output_port.py`**

```python
from abc import ABC, abstractmethod
from typing import List
from src.domain.entities.book import Book

class BookOutputPort(ABC):
    """Port de sortie pour la présentation des résultats des cas d'utilisation."""
    @abstractmethod
    def present_books(self, books: List[Book]) -> None:
        pass

    @abstractmethod
    def present_book(self, book: Book) -> None:
        pass

    @abstractmethod
    def present_error(self, message: str) -> None:
        pass
```


**`src/application/usecases/add_book_use_case.py`**

```python
import uuid
from src.domain.entities.book import Book
from src.domain.ports.input.add_book_input_port import AddBookInputPort
from src.domain.ports.output.book_repository_port import BookRepositoryPort
from src.domain.ports.output.book_output_port import BookOutputPort
from typing import Dict, Any

class AddBookUseCase(AddBookInputPort):
    """Cas d'utilisation pour ajouter un nouveau livre."""
    def __init__(self, repository: BookRepositoryPort, presenter: BookOutputPort):
        self.repository = repository
        self.presenter = presenter

    def add_book(self, book_data: Dict[str, Any]) -> None:
        # Vérification de l'existence de l'ISBN
        if self.repository.find_by_isbn(book_data.get('isbn', '')):
            self.presenter.present_error(f"Un livre avec l'ISBN {book_data['isbn']} existe déjà.")
            return

        book = Book(
            id=str(uuid.uuid4()), # Génération d'un ID unique
            title=book_data.get('title', ''),
            author=book_data.get('author', ''),
            isbn=book_data.get('isbn', '')
        )
        if not book.title or not book.author or not book.isbn:
            self.presenter.present_error("Le titre, l'auteur et l'ISBN sont obligatoires.")
            return

        saved_book = self.repository.save(book)
        self.presenter.present_book(saved_book)
```


**`src/application/usecases/list_books_use_case.py`**

```python
from src.domain.ports.input.list_books_input_port import ListBooksInputPort
from src.domain.ports.output.book_repository_port import BookRepositoryPort
from src.domain.ports.output.book_output_port import BookOutputPort

class ListBooksUseCase(ListBooksInputPort):
    """Cas d'utilisation pour lister tous les livres."""
    def __init__(self, repository: BookRepositoryPort, presenter: BookOutputPort):
        self.repository = repository
        self.presenter = presenter

    def list_books(self) -> None:
        books = self.repository.find_all()
        self.presenter.present_books(books)
```


**`src/application/usecases/get_book_use_case.py`**

```python
from src.domain.ports.input.get_book_input_port import GetBookInputPort
from src.domain.ports.output.book_repository_port import BookRepositoryPort
from src.domain.ports.output.book_output_port import BookOutputPort

class GetBookUseCase(GetBookInputPort):
    """Cas d'utilisation pour récupérer un livre par son ID."""
    def __init__(self, repository: BookRepositoryPort, presenter: BookOutputPort):
        self.repository = repository
        self.presenter = presenter

    def get_book_by_id(self, book_id: str) -> None:
        book = self.repository.find_by_id(book_id)
        if book:
            self.presenter.present_book(book)
        else:
            self.presenter.present_error(f"Livre avec l'ID {book_id} non trouvé.")
```


**`src/infrastructure/adapters/repositories/in_memory_book_repository_adapter.py`**

```python
from typing import List, Optional
from src.domain.entities.book import Book
from src.domain.ports.output.book_repository_port import BookRepositoryPort

class InMemoryBookRepositoryAdapter(BookRepositoryPort):
    """Adaptateur de persistance en mémoire pour les livres."""
    def __init__(self):
        self._books: Dict[str, Book] = {} # Utilise un dictionnaire pour un accès rapide par ID
        self._isbn_map: Dict[str, str] = {} # Pour vérifier l'unicité de l'ISBN

    def save(self, book: Book) -> Book:
        if book.id is None:
            raise ValueError("L'ID du livre doit être défini avant la sauvegarde.")
        
        if book.isbn in self._isbn_map and self._isbn_map[book.isbn] != book.id:
            raise ValueError(f"L'ISBN {book.isbn} existe déjà pour un autre livre.")
            
        self._books[book.id] = book
        self._isbn_map[book.isbn] = book.id
        return book

    def find_all(self) -> List[Book]:
        return list(self._books.values())

    def find_by_id(self, book_id: str) -> Optional[Book]:
        return self._books.get(book_id)

    def find_by_isbn(self, isbn: str) -> Optional[Book]:
        book_id = self._isbn_map.get(isbn)
        return self._books.get(book_id) if book_id else None
```


**`src/infrastructure/adapters/presenters/book_console_presenter.py`**

```python
from typing import List
from src.domain.entities.book import Book
from src.domain.ports.output.book_output_port import BookOutputPort

class BookConsolePresenter(BookOutputPort):
    """Adaptateur de présentation pour la console."""
    def present_books(self, books: List[Book]) -> None:
        if not books:
            print("Aucun livre enregistré.")
            return
        print("\n--- Liste des Livres ---")
        for book in books:
            print(f"ID: {book.id}, Titre: {book.title}, Auteur: {book.author}, ISBN: {book.isbn}")
        print("------------------------")

    def present_book(self, book: Book) -> None:
        print("\n--- Détails du Livre ---")
        print(f"ID: {book.id}")
        print(f"Titre: {book.title}")
        print(f"Auteur: {book.author}")
        print(f"ISBN: {book.isbn}")
        print("------------------------")

    def present_error(self, message: str) -> None:
        print(f"\nERREUR: {message}")
```


**`src/infrastructure/adapters/controllers/book_controller.py`**

```python
from src.domain.ports.input.add_book_input_port import AddBookInputPort
from src.domain.ports.input.list_books_input_port import ListBooksInputPort
from src.domain.ports.input.get_book_input_port import GetBookInputPort
from typing import Dict, Any

class BookController:
    """Adaptateur de contrôleur pour les opérations sur les livres."""
    def __init__(
        self,
        add_book_use_case: AddBookInputPort,
        list_books_use_case: ListBooksInputPort,
        get_book_use_case: GetBookInputPort
    ):
        self.add_book_use_case = add_book_use_case
        self.list_books_use_case = list_books_use_case
        self.get_book_use_case = get_book_use_case

    def add_book(self, book_data: Dict[str, Any]) -> None:
        self.add_book_use_case.add_book(book_data)

    def list_books(self) -> None:
        self.list_books_use_case.list_books()

    def get_book_by_id(self, book_id: str) -> None:
        self.get_book_use_case.get_book_by_id(book_id)
```


**`src/infrastructure/main/app.py`**

```python
from src.infrastructure.adapters.repositories.in_memory_book_repository_adapter import InMemoryBookRepositoryAdapter
from src.infrastructure.adapters.presenters.book_console_presenter import BookConsolePresenter
from src.application.usecases.add_book_use_case import AddBookUseCase
from src.application.usecases.list_books_use_case import ListBooksUseCase
from src.application.usecases.get_book_use_case import GetBookUseCase
from src.infrastructure.adapters.controllers.book_controller import BookController

def main():
    """Point d'entrée de l'application : assemblage des dépendances."""
    # Infrastructure
    book_repository = InMemoryBookRepositoryAdapter()
    book_presenter = BookConsolePresenter()

    # Application (Use Cases)
    add_book_use_case = AddBookUseCase(book_repository, book_presenter)
    list_books_use_case = ListBooksUseCase(book_repository, book_presenter)
    get_book_use_case = GetBookUseCase(book_repository, book_presenter)

    # Infrastructure (Controller)
    book_controller = BookController(
        add_book_use_case,
        list_books_use_case,
        get_book_use_case
    )

    # Simulation des interactions utilisateur
    print("--- Démarrage de l'application de gestion de livres ---")

    # Ajouter des livres
    book_controller.add_book({"title": "Clean Architecture", "author": "Robert C. Martin", "isbn": "978-0134494166"})
    book_controller.add_book({"title": "Domain-Driven Design", "author": "Eric Evans", "isbn": "978-0321125217"})
    book_controller.add_book({"title": "Refactoring", "author": "Martin Fowler", "isbn": "978-0134757599"})
    book_controller.add_book({"title": "Clean Code", "author": "Robert C. Martin", "isbn": "978-0132350884"})
    
    # Tentative d'ajouter un livre avec un ISBN existant
    book_controller.add_book({"title": "Another Clean Architecture", "author": "Someone Else", "isbn": "978-0134494166"})

    # Lister tous les livres
    book_controller.list_books()

    # Récupérer un livre par ID (nécessite de connaître un ID généré)
    # Pour l'exemple, nous allons récupérer le premier livre ajouté
    first_book_id = list(book_repository._books.keys())[0] if book_repository._books else None
    if first_book_id:
        book_controller.get_book_by_id(first_book_id)
    
    # Récupérer un livre inexistant
    book_controller.get_book_by_id("non-existent-id")

    print("\n--- Fin de l'application ---")

if __name__ == "__main__":
    main()
```


#### 3. Explication de l'Approche

*   **`domain` :** Contient l'entité `Book` (pure logique métier) et les interfaces (`Ports`) qui définissent les contrats de communication. Les `Input Ports` définissent ce que l'application peut faire, et les `Output Ports` définissent ce dont l'application a besoin (persistance, présentation).
*   **`application` :** Contient les `Use Cases` qui implémentent les `Input Ports`. Ils orchestrent la logique métier en utilisant les `Output Ports` injectés. Ils n'ont aucune connaissance des détails d'implémentation.
*   **`infrastructure` :** Contient les `Adapters` qui implémentent les `Output Ports` (repository, presenter) et les `Input Ports` (controller). C'est ici que les détails techniques (persistance en mémoire, affichage console) sont gérés. Le fichier `app.py` est le point d'assemblage où toutes les dépendances sont instanciées et injectées.

Cette séparation garantit que la logique métier (Use Cases) est indépendante de l'UI, de la base de données et des frameworks, ce qui la rend hautement testable et maintenable.

---

### Chapitre 2 : Clean Architecture avec DTOs et Gestion d'Erreurs Explicite (Python)

Cette solution reprend les principes de la première mais introduit des Data Transfer Objects (DTOs) pour les entrées et sorties des cas d'utilisation, et une gestion d'erreurs plus structurée via des exceptions personnalisées. Cela rend les contrats des ports encore plus explicites et les flux de données plus clairs.

#### 1. Structure du Projet


```
src/
├── domain/
│   ├── entities/
│   │   └── book.py
│   ├── exceptions/
│   │   └── book_exceptions.py
│   └── ports/
│       ├── input/
│       │   ├── add_book_input_port.py
│       │   ├── list_books_input_port.py
│       │   └── get_book_input_port.py
│       └── output/
│           ├── book_repository_port.py
│           └── book_output_port.py
├── application/
│   ├── dtos/
│   │   └── book_dtos.py
│   └── usecases/
│       ├── add_book_use_case.py
│       ├── list_books_use_case.py
│       └── get_book_use_case.py
└── infrastructure/
    ├── adapters/
    │   ├── controllers/
    │   │   └── book_controller.py
    │   ├── presenters/
    │   │   └── book_console_presenter.py
    │   └── repositories/
    │       └── in_memory_book_repository_adapter.py
    └── main/
        └── app.py
```


#### 2. Code Source

**`src/domain/entities/book.py`**

```python
from dataclasses import dataclass
from typing import Optional

@dataclass(frozen=True) # Rendre l'entité immuable est une bonne pratique
class Book:
    """Représente l'entité métier Livre."""
    id: Optional[str]
    title: str
    author: str
    isbn: str
```


**`src/domain/exceptions/book_exceptions.py`**

```python
class BookError(Exception):
    """Classe de base pour les exceptions liées aux livres."""
    pass

class BookNotFoundError(BookError):
    """Exception levée quand un livre n'est pas trouvé."""
    pass

class BookAlreadyExistsError(BookError):
    """Exception levée quand un livre avec le même ISBN existe déjà."""
    pass

class InvalidBookDataError(BookError):
    """Exception levée quand les données d'un livre sont invalides."""
    pass
```


**`src/application/dtos/book_dtos.py`**

```python
from dataclasses import dataclass
from typing import List, Optional

@dataclass(frozen=True)
class AddBookRequest:
    """DTO pour la requête d'ajout de livre."""
    title: str
    author: str
    isbn: str

@dataclass(frozen=True)
class BookResponse:
    """DTO pour la réponse d'un livre."""
    id: str
    title: str
    author: str
    isbn: str

@dataclass(frozen=True)
class BookListResponse:
    """DTO pour la réponse d'une liste de livres."""
    books: List[BookResponse]

@dataclass(frozen=True)
class ErrorResponse:
    """DTO pour la réponse d'erreur."""
    message: str
    code: Optional[str] = None
```


**`src/domain/ports/input/add_book_input_port.py`**

```python
from abc import ABC, abstractmethod
from src.application.dtos.book_dtos import AddBookRequest, BookResponse

class AddBookInputPort(ABC):
    """Port d'entrée pour l'ajout d'un livre."""
    @abstractmethod
    def add_book(self, request: AddBookRequest) -> BookResponse:
        pass
```


**`src/domain/ports/input/list_books_input_port.py`**

```python
from abc import ABC, abstractmethod
from src.application.dtos.book_dtos import BookListResponse

class ListBooksInputPort(ABC):
    """Port d'entrée pour la liste des livres."""
    @abstractmethod
    def list_books(self) -> BookListResponse:
        pass
```


**`src/domain/ports/input/get_book_input_port.py`**

```python
from abc import ABC, abstractmethod
from src.application.dtos.book_dtos import BookResponse
from typing import Optional

class GetBookInputPort(ABC):
    """Port d'entrée pour la récupération d'un livre par ID."""
    @abstractmethod
    def get_book_by_id(self, book_id: str) -> Optional[BookResponse]:
        pass
```


**`src/domain/ports/output/book_repository_port.py`**

```python
from abc import ABC, abstractmethod
from typing import List, Optional
from src.domain.entities.book import Book

class BookRepositoryPort(ABC):
    """Port de sortie pour la persistance des livres."""
    @abstractmethod
    def save(self, book: Book) -> Book:
        pass

    @abstractmethod
    def find_all(self) -> List[Book]:
        pass

    @abstractmethod
    def find_by_id(self, book_id: str) -> Optional[Book]:
        pass

    @abstractmethod
    def find_by_isbn(self, isbn: str) -> Optional[Book]:
        pass
```


**`src/domain/ports/output/book_output_port.py`**

```python
from abc import ABC, abstractmethod
from src.application.dtos.book_dtos import BookResponse, BookListResponse, ErrorResponse

class BookOutputPort(ABC):
    """Port de sortie pour la présentation des résultats des cas d'utilisation."""
    @abstractmethod
    def present_books(self, response: BookListResponse) -> None:
        pass

    @abstractmethod
    def present_book(self, response: BookResponse) -> None:
        pass

    @abstractmethod
    def present_error(self, error: ErrorResponse) -> None:
        pass
```


**`src/application/usecases/add_book_use_case.py`**

```python
import uuid
from src.domain.entities.book import Book
from src.domain.ports.input.add_book_input_port import AddBookInputPort
from src.domain.ports.output.book_repository_port import BookRepositoryPort
from src.domain.ports.output.book_output_port import BookOutputPort
from src.application.dtos.book_dtos import AddBookRequest, BookResponse, ErrorResponse
from src.domain.exceptions.book_exceptions import BookAlreadyExistsError, InvalidBookDataError

class AddBookUseCase(AddBookInputPort):
    """Cas d'utilisation pour ajouter un nouveau livre."""
    def __init__(self, repository: BookRepositoryPort, presenter: BookOutputPort):
        self.repository = repository
        self.presenter = presenter

    def add_book(self, request: AddBookRequest) -> BookResponse:
        try:
            if not request.title or not request.author or not request.isbn:
                raise InvalidBookDataError("Le titre, l'auteur et l'ISBN sont obligatoires.")

            if self.repository.find_by_isbn(request.isbn):
                raise BookAlreadyExistsError(f"Un livre avec l'ISBN {request.isbn} existe déjà.")

            book = Book(
                id=str(uuid.uuid4()),
                title=request.title,
                author=request.author,
                isbn=request.isbn
            )
            saved_book = self.repository.save(book)
            response = BookResponse(
                id=saved_book.id,
                title=saved_book.title,
                author=saved_book.author,
                isbn=saved_book.isbn
            )
            self.presenter.present_book(response)
            return response
        except (BookAlreadyExistsError, InvalidBookDataError) as e:
            self.presenter.present_error(ErrorResponse(message=str(e), code="BUSINESS_RULE_VIOLATION"))
            raise # Re-lancer pour que le contrôleur puisse aussi gérer si nécessaire
        except Exception as e:
            self.presenter.present_error(ErrorResponse(message="Une erreur inattendue est survenue.", code="UNKNOWN_ERROR"))
            raise
```


**`src/application/usecases/list_books_use_case.py`**

```python
from src.domain.ports.input.list_books_input_port import ListBooksInputPort
from src.domain.ports.output.book_repository_port import BookRepositoryPort
from src.domain.ports.output.book_output_port import BookOutputPort
from src.application.dtos.book_dtos import BookListResponse, BookResponse, ErrorResponse

class ListBooksUseCase(ListBooksInputPort):
    """Cas d'utilisation pour lister tous les livres."""
    def __init__(self, repository: BookRepositoryPort, presenter: BookOutputPort):
        self.repository = repository
        self.presenter = presenter

    def list_books(self) -> BookListResponse:
        try:
            books = self.repository.find_all()
            book_responses = [
                BookResponse(id=b.id, title=b.title, author=b.author, isbn=b.isbn)
                for b in books
            ]
            response = BookListResponse(books=book_responses)
            self.presenter.present_books(response)
            return response
        except Exception as e:
            self.presenter.present_error(ErrorResponse(message="Erreur lors de la récupération des livres.", code="REPOSITORY_ERROR"))
            raise
```


**`src/application/usecases/get_book_use_case.py`**

```python
from src.domain.ports.input.get_book_input_port import GetBookInputPort
from src.domain.ports.output.book_repository_port import BookRepositoryPort
from src.domain.ports.output.book_output_port import BookOutputPort
from src.application.dtos.book_dtos import BookResponse, ErrorResponse
from src.domain.exceptions.book_exceptions import BookNotFoundError
from typing import Optional

class GetBookUseCase(GetBookInputPort):
    """Cas d'utilisation pour récupérer un livre par son ID."""
    def __init__(self, repository: BookRepositoryPort, presenter: BookOutputPort):
        self.repository = repository
        self.presenter = presenter

    def get_book_by_id(self, book_id: str) -> Optional[BookResponse]:
        try:
            book = self.repository.find_by_id(book_id)
            if not book:
                raise BookNotFoundError(f"Livre avec l'ID {book_id} non trouvé.")
            
            response = BookResponse(
                id=book.id,
                title=book.title,
                author=book.author,
                isbn=book.isbn
            )
            self.presenter.present_book(response)
            return response
        except BookNotFoundError as e:
            self.presenter.present_error(ErrorResponse(message=str(e), code="NOT_FOUND"))
            return None
        except Exception as e:
            self.presenter.present_error(ErrorResponse(message="Erreur lors de la récupération du livre.", code="REPOSITORY_ERROR"))
            raise
```


**`src/infrastructure/adapters/repositories/in_memory_book_repository_adapter.py`**

```python
from typing import List, Optional, Dict
from src.domain.entities.book import Book
from src.domain.ports.output.book_repository_port import BookRepositoryPort
from src.domain.exceptions.book_exceptions import BookAlreadyExistsError

class InMemoryBookRepositoryAdapter(BookRepositoryPort):
    """Adaptateur de persistance en mémoire pour les livres."""
    def __init__(self):
        self._books: Dict[str, Book] = {}
        self._isbn_map: Dict[str, str] = {}

    def save(self, book: Book) -> Book:
        if book.id is None:
            raise ValueError("L'ID du livre doit être défini avant la sauvegarde.")
        
        # Vérification de l'unicité de l'ISBN lors de la sauvegarde
        if book.isbn in self._isbn_map and self._isbn_map[book.isbn] != book.id:
            raise BookAlreadyExistsError(f"L'ISBN {book.isbn} existe déjà pour un autre livre.")
            
        self._books[book.id] = book
        self._isbn_map[book.isbn] = book.id
        return book

    def find_all(self) -> List[Book]:
        return list(self._books.values())

    def find_by_id(self, book_id: str) -> Optional[Book]:
        return self._books.get(book_id)

    def find_by_isbn(self, isbn: str) -> Optional[Book]:
        book_id = self._isbn_map.get(isbn)
        return self._books.get(book_id) if book_id else None
```


**`src/infrastructure/adapters/presenters/book_console_presenter.py`**

```python
from src.domain.ports.output.book_output_port import BookOutputPort
from src.application.dtos.book_dtos import BookResponse, BookListResponse, ErrorResponse

class BookConsolePresenter(BookOutputPort):
    """Adaptateur de présentation pour la console."""
    def present_books(self, response: BookListResponse) -> None:
        if not response.books:
            print("Aucun livre enregistré.")
            return
        print("\n--- Liste des Livres ---")
        for book in response.books:
            print(f"ID: {book.id}, Titre: {book.title}, Auteur: {book.author}, ISBN: {book.isbn}")
        print("------------------------")

    def present_book(self, response: BookResponse) -> None:
        print("\n--- Détails du Livre ---")
        print(f"ID: {response.id}")
        print(f"Titre: {response.title}")
        print(f"Auteur: {response.author}")
        print(f"ISBN: {response.isbn}")
        print("------------------------")

    def present_error(self, error: ErrorResponse) -> None:
        print(f"\nERREUR ({error.code or 'UNKNOWN'}): {error.message}")
```


**`src/infrastructure/adapters/controllers/book_controller.py`**

```python
from src.domain.ports.input.add_book_input_port import AddBookInputPort
from src.domain.ports.input.list_books_input_port import ListBooksInputPort
from src.domain.ports.input.get_book_input_port import GetBookInputPort
from src.application.dtos.book_dtos import AddBookRequest
from src.domain.exceptions.book_exceptions import BookError

class BookController:
    """Adaptateur de contrôleur pour les opérations sur les livres."""
    def __init__(
        self,
        add_book_use_case: AddBookInputPort,
        list_books_use_case: ListBooksInputPort,
        get_book_use_case: GetBookInputPort
    ):
        self.add_book_use_case = add_book_use_case
        self.list_books_use_case = list_books_use_case
        self.get_book_use_case = get_book_use_case

    def add_book(self, title: str, author: str, isbn: str) -> None:
        request = AddBookRequest(title=title, author=author, isbn=isbn)
        try:
            self.add_book_use_case.add_book(request)
        except BookError:
            # L'erreur est déjà présentée par le Use Case via le Presenter
            pass

    def list_books(self) -> None:
        try:
            self.list_books_use_case.list_books()
        except BookError:
            pass

    def get_book_by_id(self, book_id: str) -> None:
        try:
            self.get_book_use_case.get_book_by_id(book_id)
        except BookError:
            pass
```


**`src/infrastructure/main/app.py`**

```python
from src.infrastructure.adapters.repositories.in_memory_book_repository_adapter import InMemoryBookRepositoryAdapter
from src.infrastructure.adapters.presenters.book_console_presenter import BookConsolePresenter
from src.application.usecases.add_book_use_case import AddBookUseCase
from src.application.usecases.list_books_use_case import ListBooksUseCase
from src.application.usecases.get_book_use_case import GetBookUseCase
from src.infrastructure.adapters.controllers.book_controller import BookController

def main():
    """Point d'entrée de l'application : assemblage des dépendances."""
    # Infrastructure
    book_repository = InMemoryBookRepositoryAdapter()
    book_presenter = BookConsolePresenter()

    # Application (Use Cases)
    add_book_use_case = AddBookUseCase(book_repository, book_presenter)
    list_books_use_case = ListBooksUseCase(book_repository, book_presenter)
    get_book_use_case = GetBookUseCase(book_repository, book_presenter)

    # Infrastructure (Controller)
    book_controller = BookController(
        add_book_use_case,
        list_books_use_case,
        get_book_use_case
    )

    # Simulation des interactions utilisateur
    print("--- Démarrage de l'application de gestion de livres ---")

    # Ajouter des livres
    book_controller.add_book("Clean Architecture", "Robert C. Martin", "978-0134494166")
    book_controller.add_book("Domain-Driven Design", "Eric Evans", "978-0321125217")
    book_controller.add_book("Refactoring", "Martin Fowler", "978-0134757599")
    book_controller.add_book("Clean Code", "Robert C. Martin", "978-0132350884")
    
    # Tentative d'ajouter un livre avec un ISBN existant
    book_controller.add_book("Another Clean Architecture", "Someone Else", "978-0134494166")
    
    # Tentative d'ajouter un livre avec des données invalides
    book_controller.add_book("", "Invalid Author", "123-INVALID")

    # Lister tous les livres
    book_controller.list_books()

    # Récupérer un livre par ID (nécessite de connaître un ID généré)
    # Pour l'exemple, nous allons récupérer le premier livre ajouté
    first_book_id = list(book_repository._books.keys())[0] if book_repository._books else None
    if first_book_id:
        book_controller.get_book_by_id(first_book_id)
    
    # Récupérer un livre inexistant
    book_controller.get_book_by_id("non-existent-id")

    print("\n--- Fin de l'application ---")

if __name__ == "__main__":
    main()
```


#### 3. Explication de l'Approche

Cette solution affine la précédente en introduisant des DTOs et une gestion d'erreurs plus robuste :

*   **DTOs (`application/dtos`) :** Les objets `AddBookRequest`, `BookResponse`, `BookListResponse`, `ErrorResponse` sont utilisés pour définir explicitement les données qui traversent les frontières des couches. Les `Input Ports` acceptent des `Request` DTOs et les `Output Ports` reçoivent des `Response` DTOs. Cela rend les contrats plus clairs et facilite la validation des données d'entrée/sortie.
*   **Exceptions personnalisées (`domain/exceptions`) :** Des exceptions spécifiques au domaine (`BookNotFoundError`, `BookAlreadyExistsError`, `InvalidBookDataError`) sont définies. Les `Use Cases` lèvent ces exceptions en cas de violation des règles métier ou de problèmes de données. Le `Presenter` est ensuite chargé de les formater en `ErrorResponse` pour l'affichage.
*   **Immuabilité :** L'entité `Book` est marquée comme `frozen=True` pour la rendre immuable, ce qui peut simplifier la logique et prévenir des modifications inattendues.
*   **Gestion des erreurs dans les Use Cases :** Les `Use Cases` encapsulent la logique de gestion des erreurs métier. Ils attrapent les exceptions du domaine et utilisent le `BookOutputPort` pour présenter l'erreur. Ils peuvent également re-lancer l'exception pour permettre au contrôleur de réagir si nécessaire (par exemple, pour renvoyer un code HTTP spécifique dans une API REST).

Cette approche, bien que légèrement plus verbeuse, offre une meilleure clarté des flux de données, une gestion des erreurs plus précise et une séparation encore plus rigoureuse des préoccupations, ce qui est bénéfique pour des applications plus complexes.