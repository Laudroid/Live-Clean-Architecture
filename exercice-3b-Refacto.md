---

# TP Architecture Logicielle : Analyse Critique et Refactorisation selon le DIP

## Objectif du TP
- Comprendre et identifier un couplage fort dans un code existant.
- Appliquer le principe d'inversion des dépendances (DIP) pour réaliser une refactorisation.
- Développer un regard critique sur la conception logicielle existante.
- Exploiter intelligemment un assistant d’IA pour aider à la compréhension et à la refactorisation du code.

---

## Contexte et mini-projet

Vous travaillez sur un mini-projet simple de gestion d’envoi de notifications dans une application. Cette application peut envoyer des notifications par email ou SMS.

Voici une version initiale, simpliste et fortement couplée de la classe `NotificationSender` :

```python
class EmailService:
    def send_email(self, recipient, message):
        print(f"Envoi d'un email à {recipient} : {message}")

class SmsService:
    def send_sms(self, phone_number, message):
        print(f"Envoi d'un SMS à {phone_number} : {message}")

class NotificationSender:
    def __init__(self):
        self.email_service = EmailService()
        self.sms_service = SmsService()

    def send_notification(self, contact_info, message, channel):
        if channel == 'email':
            self.email_service.send_email(contact_info, message)
        elif channel == 'sms':
            self.sms_service.send_sms(contact_info, message)
```

---

## Énoncé du TP

### 1. Analyse critique

- Identifiez et décrivez le ou les problèmes d’architecture et de couplage dans cet exemple.
- Expliquez pourquoi ce code ne respecte pas le **DIP (Dependency Inversion Principle)**.
- Quelles sont les conséquences possibles de ce couplage fort sur l’évolution future de l’application ?

### 2. Refactorisation

- Refactorez ce code pour que `NotificationSender` respecte le DIP.
- Introduisez des abstractions (interfaces ou classes abstraites) pour découpler `NotificationSender` des classes concrètes `EmailService` et `SmsService`.
- Permettez à `NotificationSender` de pouvoir utiliser n’importe quel service d’envoi de notification, sans modification du code source.

### 3. Extension (facultatif)

- Ajoutez un nouveau type de service de notification (par exemple, `PushNotificationService`).
- Montrez que vous pouvez facilement l’intégrer dans votre version refactorée sans modifier la classe `NotificationSender`.

---