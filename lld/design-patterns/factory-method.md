# Factory Method: LLD interview quick reference

## What problem does Factory Method solve?

Imagine a notification system that starts with email, then later needs SMS, push, Slack, and WhatsApp notifications.

The service should **send** a notification. It should not need to know the concrete class or setup needed to **create** every notification type.

### The problem: client code chooses concrete classes

```cpp
// Hard to extend: every new type requires modifying this method.
void sendNotification(const std::string& type, const std::string& message) {
    if (type == "email") {
        EmailNotification notification("smtp.gmail.com");
        notification.send(message);
    } else if (type == "sms") {
        SMSNotification notification("telephony-api-key");
        notification.send(message);
    }
}
```

Problems:

- This method knows every concrete notification class.
- Adding `PushNotification` requires editing the same `if / else if` chain.
- Construction/configuration logic becomes repeated across the application.
- The code violates the Open/Closed Principle: adding a type modifies existing core logic.

### The idea

Factory Method moves the decision of **which product to create** into subclasses/implementations called **creators**.

```text
Client → ConcreteCreator → ConcreteProduct
           ↑                    ↑
           │                    │
        Creator              Product interface
```

> Interview sentence: **Factory Method defines an interface for creating a product, while subclasses decide which concrete product to create.**

---

## Four roles to remember

| Role | Notification example | Responsibility |
| --- | --- | --- |
| Product | `Notification` | Common interface for objects being created. |
| Concrete Product | `EmailNotification`, `SMSNotification` | Real implementations of the product. |
| Creator | `NotificationCreator` | Declares the factory method and often owns the common workflow. |
| Concrete Creator | `EmailNotificationCreator`, `SMSNotificationCreator` | Overrides the factory method to construct one concrete product. |

The factory method itself is normally named something like `createNotification()`.

---

## C++: Factory Method with `std::unique_ptr`

```cpp
#include <iostream>
#include <memory>
#include <string>
#include <utility>

// PRODUCT: all notification types must provide send().
class Notification {
public:
    virtual void send(const std::string& message) = 0;

    // Required when deleting a derived object through Notification*.
    virtual ~Notification() = default;
};

// CONCRETE PRODUCT: one way to send a notification.
class EmailNotification : public Notification {
public:
    // `explicit` prevents accidental conversion from std::string.
    explicit EmailNotification(std::string smtpServer)
        : smtpServer_(std::move(smtpServer)) {}

    void send(const std::string& message) override {
        std::cout << "Email via " << smtpServer_ << ": " << message << '\n';
    }

private:
    std::string smtpServer_;
};

// CONCRETE PRODUCT: another way to send a notification.
class SMSNotification : public Notification {
public:
    explicit SMSNotification(std::string apiKey)
        : apiKey_(std::move(apiKey)) {}

    void send(const std::string& message) override {
        std::cout << "SMS using " << apiKey_ << ": " << message << '\n';
    }

private:
    std::string apiKey_;
};

// CREATOR: owns the common workflow but does not know the concrete product.
class NotificationCreator {
public:
    virtual ~NotificationCreator() = default;

    // Factory method: each concrete creator decides what to create.
    virtual std::unique_ptr<Notification> createNotification() const = 0;

    // Shared workflow (often called a template method):
    // create product → use product.
    void sendNotification(const std::string& message) const {
        std::unique_ptr<Notification> notification = createNotification();
        notification->send(message);
    }
};

// CONCRETE CREATOR: creates EmailNotification only.
class EmailNotificationCreator : public NotificationCreator {
public:
    explicit EmailNotificationCreator(std::string smtpServer)
        : smtpServer_(std::move(smtpServer)) {}

    std::unique_ptr<Notification> createNotification() const override {
        // Caller receives the product through the Notification interface.
        return std::make_unique<EmailNotification>(smtpServer_);
    }

private:
    std::string smtpServer_;
};

// CONCRETE CREATOR: creates SMSNotification only.
class SMSNotificationCreator : public NotificationCreator {
public:
    explicit SMSNotificationCreator(std::string apiKey)
        : apiKey_(std::move(apiKey)) {}

    std::unique_ptr<Notification> createNotification() const override {
        return std::make_unique<SMSNotification>(apiKey_);
    }

private:
    std::string apiKey_;
};

int main() {
    // Client selects a creator from configuration or business rules.
    std::unique_ptr<NotificationCreator> creator =
        std::make_unique<EmailNotificationCreator>("smtp.gmail.com");

    // Client uses the shared workflow. It does not construct EmailNotification.
    creator->sendNotification("Welcome, Asha!");
}
```

Output:

```text
Email via smtp.gmail.com: Welcome, Asha!
```

### C++ flow, line by line

```text
creator->sendNotification("Welcome")
  → NotificationCreator::sendNotification runs
  → it calls createNotification()
  → virtual dispatch selects EmailNotificationCreator::createNotification()
  → EmailNotification is created and returned as unique_ptr<Notification>
  → notification->send("Welcome") calls EmailNotification::send()
```

### C++ valid and invalid usage

```cpp
// VALID: client depends on the base Creator interface.
void notifyUser(const NotificationCreator& creator) {
    creator.sendNotification("Your order was shipped");
}

EmailNotificationCreator emailCreator("smtp.gmail.com");
notifyUser(emailCreator);

// INVALID: Notification is abstract because send() = 0.
// Notification notification;

// INVALID: unique_ptr cannot be copied; it has one owner.
// auto secondOwner = creator;

// VALID: move ownership if needed.
// auto secondOwner = std::move(creator);
```

### C++ facts to say in an interview

- `Notification` is the Product interface; `EmailNotification` is a Concrete Product.
- `createNotification()` is the Factory Method.
- `NotificationCreator` contains common workflow; Concrete Creators choose the product.
- `std::unique_ptr<Notification>` clearly expresses ownership and automatically deletes the created object.
- To add `PushNotification`, add `PushNotification` and `PushNotificationCreator`; existing creator classes stay unchanged.

Compile:

```bash
g++ -std=c++17 factory-method.cpp -o factory-method
./factory-method
```

---

## Python: Factory Method with abstract base classes

```python
from abc import ABC, abstractmethod


# PRODUCT: common contract.
class Notification(ABC):
    @abstractmethod
    def send(self, message):
        pass


# CONCRETE PRODUCT.
class EmailNotification(Notification):
    def __init__(self, smtp_server):
        self.smtp_server = smtp_server

    def send(self, message):
        print(f"Email via {self.smtp_server}: {message}")


# CONCRETE PRODUCT.
class SMSNotification(Notification):
    def __init__(self, api_key):
        self.api_key = api_key

    def send(self, message):
        print(f"SMS using {self.api_key}: {message}")


# CREATOR: has shared workflow and declares the factory method.
class NotificationCreator(ABC):
    @abstractmethod
    def create_notification(self):
        """Factory method: subclasses return one concrete Notification."""
        pass

    def send_notification(self, message):
        # Shared workflow: creation is delegated, usage stays here.
        notification = self.create_notification()
        notification.send(message)


# CONCRETE CREATOR.
class EmailNotificationCreator(NotificationCreator):
    def __init__(self, smtp_server):
        self.smtp_server = smtp_server

    def create_notification(self):
        return EmailNotification(self.smtp_server)


# CONCRETE CREATOR.
class SMSNotificationCreator(NotificationCreator):
    def __init__(self, api_key):
        self.api_key = api_key

    def create_notification(self):
        return SMSNotification(self.api_key)


# Client selects one creator.
creator = EmailNotificationCreator("smtp.gmail.com")

# Client uses shared behavior without constructing EmailNotification directly.
creator.send_notification("Welcome, Asha!")
```

### Python valid and invalid usage

```python
def notify_user(creator: NotificationCreator):
    creator.send_notification("Your order was shipped")


notify_user(EmailNotificationCreator("smtp.gmail.com"))  # Valid

# Invalid: abstract classes cannot be created directly.
# Notification()
# NotificationCreator()
```

### Python interview note

- Python does not enforce interfaces as strictly as C++, but `ABC` and `@abstractmethod` express the design clearly.
- Duck typing is also common: if objects have compatible `send` methods, Python may not need the abstract base class.
- The Factory Method idea remains the same: the creator chooses the concrete product.

---

## Go: Factory Method with interfaces

Go does not use inheritance. It uses interfaces and structs. The shared workflow is normally a function that accepts the creator interface.

```go
package main

import "fmt"

// PRODUCT interface.
type Notification interface {
	Send(message string)
}

// CONCRETE PRODUCT.
type EmailNotification struct {
	SMTPServer string
}

func (notification *EmailNotification) Send(message string) {
	fmt.Printf("Email via %s: %s\n", notification.SMTPServer, message)
}

// CONCRETE PRODUCT.
type SMSNotification struct {
	APIKey string
}

func (notification *SMSNotification) Send(message string) {
	fmt.Printf("SMS using %s: %s\n", notification.APIKey, message)
}

// CREATOR interface: declares the factory method.
type NotificationCreator interface {
	CreateNotification() Notification
}

// CONCRETE CREATOR.
type EmailNotificationCreator struct {
	SMTPServer string
}

func (creator EmailNotificationCreator) CreateNotification() Notification {
	return &EmailNotification{SMTPServer: creator.SMTPServer}
}

// CONCRETE CREATOR.
type SMSNotificationCreator struct {
	APIKey string
}

func (creator SMSNotificationCreator) CreateNotification() Notification {
	return &SMSNotification{APIKey: creator.APIKey}
}

// Shared workflow. Go uses a function instead of an inherited base method.
func SendNotification(creator NotificationCreator, message string) {
	notification := creator.CreateNotification()
	notification.Send(message)
}

func main() {
	creator := EmailNotificationCreator{SMTPServer: "smtp.gmail.com"}
	SendNotification(creator, "Welcome, Asha!")
}
```

### Go valid and invalid usage

```go
// Valid: EmailNotificationCreator satisfies NotificationCreator because it has
// CreateNotification() Notification.
creator := EmailNotificationCreator{SMTPServer: "smtp.gmail.com"}
SendNotification(creator, "Your order was shipped")

// Invalid: a string does not implement NotificationCreator.
// SendNotification("email", "Your order was shipped")
```

### Go facts to say in an interview

- Interfaces are satisfied implicitly: no `implements` keyword is needed.
- `EmailNotificationCreator` is a `NotificationCreator` because its method matches the interface exactly.
- The common workflow is usually a standalone function because Go has no base-class implementation to inherit.
- A Go constructor is usually an ordinary `NewThing(...)` function; use one when validation or defaults are needed.

Run:

```bash
go run factory-method.go
```

---

## Factory Method versus Simple Factory

| Approach | Where is “which product?” chosen? | Tradeoff |
| --- | --- | --- |
| Direct `if / else` | Client/service | Fast to start, but client becomes tightly coupled. |
| Simple Factory | One central factory with `if / else` or `switch` | Centralizes creation, but new products still modify that factory. |
| Factory Method | Concrete creator subclasses/implementations | More classes, but adding a product can be an extension rather than a change. |

Simple Factory is useful in real code, but it is not the formal Gang of Four Factory Method pattern.

## When to use—or avoid—Factory Method

Use it when:

- The concrete type is selected at runtime from configuration, user choice, or business rules.
- Creation has setup logic you want to keep out of client code.
- Several products share an interface but behave differently.
- You expect new product types and want extension points.

Avoid it when:

- There is only one product and no realistic need for variants.
- A small switch is stable and simpler than several creator classes.
- The extra Creator classes would add boilerplate with no benefit.

> Good interview answer: “Factory Method separates product creation from product usage. The creator owns the workflow, while concrete creators decide which concrete product is built.”
