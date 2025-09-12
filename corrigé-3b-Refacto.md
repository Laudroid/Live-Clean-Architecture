
# Correction TP – Architecture Logicielle & DIP

## 1. Analyse critique

### Problèmes identifiés

* **Couplage fort** : `NotificationSender` instancie directement `EmailService` et `SmsService`.
* **Responsabilité multiple** : la classe `NotificationSender` gère à la fois la logique métier (envoyer une notification) et la création/gestion des services de notification.
* **Violation du DIP** : la classe de haut niveau (`NotificationSender`) dépend de classes de bas niveau (`EmailService`, `SmsService`) au lieu de dépendre d’abstractions.

### Pourquoi ça ne respecte pas le DIP

Le **DIP (Dependency Inversion Principle)** dit :

* Les modules de haut niveau ne doivent pas dépendre des modules de bas niveau.
* Tous doivent dépendre d’abstractions.

Ici, `NotificationSender` (haut niveau) dépend **directement** de `EmailService` et `SmsService` (bas niveau).
Il n’y a **aucune abstraction** qui définit un contrat générique de "service de notification".

### Conséquences possibles

* **Difficulté d’évolution** : ajouter un nouveau type de notification oblige à modifier `NotificationSender` → risque d’erreurs, manque d’extensibilité.
* **Faible testabilité** : difficile de tester `NotificationSender` indépendamment, car il instancie directement les services concrets.
* **Manque de flexibilité** : impossible d’injecter dynamiquement un service de notification (ex. en configuration ou en test).

---

## 2. Refactorisation (Respect du DIP)

### Étape 1 : introduire une abstraction

On définit une interface (ou classe abstraite) `INotificationService` :

```python
from abc import ABC, abstractmethod

class INotificationService(ABC):
    @abstractmethod
    def send(self, recipient, message):
        pass
```

### Étape 2 : adapter les services existants

```python
class EmailService(INotificationService):
    def send(self, recipient, message):
        print(f"Envoi d'un email à {recipient} : {message}")

class SmsService(INotificationService):
    def send(self, recipient, message):
        print(f"Envoi d'un SMS à {recipient} : {message}")
```

### Étape 3 : injecter les dépendances dans `NotificationSender`

```python
class NotificationSender:
    def __init__(self, service: INotificationService):
        self.service = service   # Injection de dépendance

    def send_notification(self, recipient, message):
        self.service.send(recipient, message)
```

### Étape 4 : utilisation

```python
# Envoi par email
email_sender = NotificationSender(EmailService())
email_sender.send_notification("alice@example.com", "Hello Alice !")

# Envoi par SMS
sms_sender = NotificationSender(SmsService())
sms_sender.send_notification("0612345678", "Salut Bob !")
```

Maintenant, `NotificationSender` dépend **d’une abstraction** (`INotificationService`) et non plus des classes concrètes.
On respecte bien le DIP.

---

## 3. Extension (facultatif)

Ajout d’un nouveau service `PushNotificationService` :

```python
class PushNotificationService(INotificationService):
    def send(self, recipient, message):
        print(f"Envoi d'une notification push à {recipient} : {message}")
```

Utilisation sans modifier `NotificationSender` :

```python
push_sender = NotificationSender(PushNotificationService())
push_sender.send_notification("user123", "Nouvelle alerte disponible !")
```

Grâce au DIP, **ajouter un nouveau canal de notification ne nécessite aucune modification dans `NotificationSender`**.

---
