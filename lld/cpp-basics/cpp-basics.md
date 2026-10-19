# C++ basics: interview quick reference

This note collects small but important C++ concepts for quick revision.

## `explicit`: prevent surprising automatic conversions

### The problem

A constructor that can be called with **one argument** can also become an automatic conversion path unless it is marked `explicit`.

```cpp
#include <string>

class EmailNotification {
public:
    // Without `explicit`, a std::string can silently become an EmailNotification.
    EmailNotification(const std::string& smtpServer)
        : smtpServer_(smtpServer) {}

private:
    std::string smtpServer_;
};
```

```cpp
std::string smtpServer = "smtp.gmail.com";

// Copy initialization. This is allowed above because C++ silently calls:
// EmailNotification(smtpServer)
EmailNotification notification = smtpServer;
```

That conversion might be valid, but it is easy to miss when reading code. A string and an `EmailNotification` are different concepts:

```text
std::string              EmailNotification
"smtp.gmail.com"   →    an object that knows how to send email
```

### The safer version

```cpp
#include <string>

class EmailNotification {
public:
    // `explicit` forbids implicit conversion from std::string.
    explicit EmailNotification(const std::string& smtpServer)
        : smtpServer_(smtpServer) {}

private:
    std::string smtpServer_;
};
```

```cpp
std::string smtpServer = "smtp.gmail.com";

// EmailNotification notification = smtpServer;
// Error: C++ will not silently convert std::string to EmailNotification.

// Direct initialization: clearly says “construct an EmailNotification”.
EmailNotification notification(smtpServer);

// Brace initialization: equally clear.
EmailNotification anotherNotification{smtpServer};
```

### Why it matters in functions

Without `explicit`, a function that expects a fully constructed notification could accidentally accept a string:

```cpp
void addNotificationChannel(EmailNotification notification);

std::string smtpServer = "smtp.gmail.com";

// Without `explicit`: allowed; C++ silently constructs EmailNotification.
addNotificationChannel(smtpServer);
```

With `explicit`, the caller must make the conversion visible:

```cpp
addNotificationChannel(EmailNotification(smtpServer));
```

This is clearer for both the compiler and the person reading the code.

### Direct initialization vs copy initialization

```cpp
std::string smtpServer = "smtp.gmail.com";

// Copy initialization: can use an implicit converting constructor.
EmailNotification a = smtpServer;

// Direct initialization: directly calls the constructor.
EmailNotification b(smtpServer);

// Direct-list initialization: directly calls the constructor.
EmailNotification c{smtpServer};
```

When the constructor is `explicit`:

| Code | Result when the constructor is `explicit` | Why |
| --- | --- | --- |
| `EmailNotification a = smtpServer;` | **Compiler error** | Copy initialization would need an implicit conversion. |
| `EmailNotification b(smtpServer);` | **Valid** | Direct initialization visibly calls the constructor. |
| `EmailNotification c{smtpServer};` | **Valid** | Direct-list initialization visibly calls the constructor. |

### Complete valid/invalid example

```cpp
#include <string>

class EmailNotification {
public:
    explicit EmailNotification(const std::string& smtpServer)
        : smtpServer_(smtpServer) {}

private:
    std::string smtpServer_;
};

void addNotificationChannel(EmailNotification notification) {
    // Store or use the complete EmailNotification object.
}

int main() {
    std::string smtpServer = "smtp.gmail.com";

    // INVALID: compiler error.
    // A std::string is not automatically an EmailNotification.
    // EmailNotification first = smtpServer;

    // VALID: direct construction is clear.
    EmailNotification second(smtpServer);

    // VALID: brace construction is clear too.
    EmailNotification third{smtpServer};

    // INVALID: compiler error.
    // The function asks for EmailNotification, not std::string.
    // addNotificationChannel(smtpServer);

    // VALID: the caller explicitly constructs the required type.
    addNotificationChannel(EmailNotification{smtpServer});
}
```

### Rule of thumb

> Mark a constructor `explicit` when it takes one argument, unless automatic conversion is clearly useful and natural.

For example, turning a server address into an `EmailNotification` should be intentional, so `explicit` is appropriate.

`explicit` is not required by Factory Method, Builder, or any other design pattern. It is a C++ language feature that makes APIs safer and clearer.

---

## RAII and smart pointers: safe resource ownership

### What is RAII?

**RAII** means **Resource Acquisition Is Initialization**.

It means: put a resource inside an object. The object's constructor acquires the resource, and its destructor releases it automatically when the object leaves scope.

```text
Enter scope                     Leave scope
    │                                │
    ▼                                ▼
object is created             object is destroyed
resource is acquired          resource is released automatically
```

Resources are not only memory. They include files, locks, database connections, sockets, and memory.

### RAII without pointers: prefer an ordinary object

```cpp
#include <iostream>
#include <string>

class File {
public:
    explicit File(const std::string& fileName) {
        std::cout << "Open: " << fileName << '\n';
    }

    ~File() {
        // Runs automatically when the File object leaves scope.
        std::cout << "Close file\n";
    }
};

int main() {
    {
        File file("notes.txt");
        // Use file here.
    } // file.~File() runs automatically here.
}
```

```text
Stack frame for main
┌─────────────────────────────────┐
│ File file("notes.txt")          │
│                                 │
│ file owns the open file resource │
└─────────────────────────────────┘
              │ scope ends
              ▼
File destructor closes the resource
```

This is the best default. No dynamic allocation is needed.

### Smart-pointer headers

```cpp
#include <memory>  // std::unique_ptr, std::shared_ptr, std::weak_ptr,
                    // std::make_unique, std::make_shared
```

`<memory>` is the main header for standard smart pointers.

---

### `std::unique_ptr`: exactly one owner

Use `std::unique_ptr<T>` when one object owns a dynamically allocated `T`.

```cpp
#include <memory>
#include <string>
#include <utility> // std::move

class EmailNotification {
public:
    explicit EmailNotification(std::string smtpServer)
        : smtpServer_(std::move(smtpServer)) {}

private:
    std::string smtpServer_;
};

int main() {
    // Creates EmailNotification on the heap.
    // unique_ptr is the one owner and will delete it automatically.
    auto notification = std::make_unique<EmailNotification>("smtp.gmail.com");

} // notification is destroyed, then EmailNotification is deleted.
```

```text
Stack                                Heap
┌───────────────────────────┐        ┌───────────────────────────┐
│ unique_ptr notification   │ ─────► │ EmailNotification object  │
│ the only owner            │        │ smtpServer = ...          │
└───────────────────────────┘        └───────────────────────────┘
            │ scope ends
            ▼
unique_ptr deletes the heap object
```

#### Valid and invalid `unique_ptr` operations

```cpp
auto first = std::make_unique<EmailNotification>("smtp.gmail.com");

// INVALID: copying would create two owners of one object.
// auto second = first;

// VALID: ownership moves from first to second.
auto second = std::move(first);

// first is now empty (it holds nullptr).
// second is now the only owner.
```

```text
Before std::move:       first ─────► object
                         second      empty

After std::move:        first        empty
                         second ─────► object
```

Use `std::move` only when you intentionally transfer ownership.

---

### `std::shared_ptr`: shared ownership

Use `std::shared_ptr<T>` only when multiple objects must jointly keep `T` alive.

```cpp
#include <memory>

auto config = std::make_shared<AppConfig>();

// Both variables share ownership of the same AppConfig object.
auto orderServiceConfig = config;
auto emailWorkerConfig = config;
```

```text
orderServiceConfig ──┐
                     ├──► AppConfig object
emailWorkerConfig ───┘       shared owner count: 2

When the last shared_ptr owner disappears:
AppConfig object is deleted.
```

Copying a `shared_ptr` is valid because it intentionally increases the shared-owner count.

```cpp
auto first = std::make_shared<AppConfig>();
auto second = first; // Valid: two shared owners.
```

Be careful: shared ownership can make object lifetime harder to reason about. Prefer `unique_ptr` unless ownership is truly shared.

---

### `std::weak_ptr`: observe without owning

`std::weak_ptr<T>` observes an object managed by `shared_ptr`, but does not keep it alive.

```cpp
#include <memory>

std::weak_ptr<AppConfig> observer;

{
    auto owner = std::make_shared<AppConfig>();
    observer = owner; // observer does not increase shared ownership.
} // owner is gone, so AppConfig is deleted.

if (auto config = observer.lock()) {
    // Valid only if the AppConfig still exists.
    // config is a temporary shared_ptr owner in this block.
} else {
    // The AppConfig no longer exists.
}
```

```text
shared_ptr owner ─────► AppConfig object
weak_ptr observer ─ ─ ─► observes it, but does not keep it alive
```

`weak_ptr` is commonly used to break circular ownership:

```text
BAD: shared_ptr A ─► B and shared_ptr B ─► A
     Neither owner count becomes zero, so neither object is deleted.

BETTER: shared_ptr A ─► B and weak_ptr B ─ ─► A
        A disappearing can now allow both objects to be cleaned up.
```

---

### Borrowing: references and raw pointers can still be correct

Smart pointers express **ownership**. A function that merely uses an object should usually not receive ownership.

```cpp
// The function borrows notification. It cannot be null.
void send(Notification& notification) {
    notification.send("Hello");
}

// The function borrows notification. nullptr is allowed and must be checked.
void sendIfPresent(Notification* notification) {
    if (notification != nullptr) {
        notification->send("Hello");
    }
}
```

```text
Owner:    unique_ptr<Notification> ─────► Notification
Borrower: Notification& or Notification* ─► same object

Borrower does NOT call delete.
Owner decides the lifetime.
```

### Raw `new` and `delete`: avoid for ordinary application code

```cpp
// Avoid: manual ownership is easy to get wrong.
EmailNotification* notification = new EmailNotification("smtp.gmail.com");

// If an exception or early return happens before this line, memory leaks.
delete notification;
```

```cpp
// Prefer: cleanup is automatic, including on exceptions and early returns.
auto notification = std::make_unique<EmailNotification>("smtp.gmail.com");
```

Use raw `new` / `delete` mainly in low-level code that implements memory management itself, such as object pools, custom allocators, or placement `new`.

### Quick ownership decision

| Need | Prefer |
| --- | --- |
| Object can live in the current scope | An ordinary object: `T object;` |
| One dynamic owner | `std::unique_ptr<T>` |
| Several real owners | `std::shared_ptr<T>` |
| Observe a `shared_ptr` object without keeping it alive | `std::weak_ptr<T>` |
| Borrow; null is not allowed | `T&` |
| Borrow; null is meaningful | `T*` |

> Memory rule: **Use a raw pointer to borrow, not to own. Use an ordinary object when possible; otherwise prefer `unique_ptr`.**
