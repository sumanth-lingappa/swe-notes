---
layout: default
title: C++ basics
---

# C++ basics: quick reference

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

---

## Type casting: converting one type to another

A **cast** tells C++ to treat or convert a value as another type.

There are two broad kinds:

```text
Implicit conversion  → C++ performs it automatically.
Explicit conversion  → You clearly request it with syntax such as static_cast<T>(value).
```

Prefer the named C++ casts. Avoid C-style casts such as `(int)value` because they hide which kind of conversion is happening.

### 1. Implicit conversion

C++ automatically converts when it considers the conversion safe or natural enough.

```cpp
int count = 42;

// int becomes double automatically.
double average = count;

// char becomes int automatically.
int asciiCode = 'A'; // 65 in common character encodings.
```

```text
int 42 ──implicit conversion──► double 42.0
```

Be careful: automatic does not always mean lossless.

```cpp
double price = 19.99;

// VALID, but data after the decimal point is discarded.
int wholePrice = price; // 19
```

### 2. Direct initialization and explicit conversion

These both make the destination type visible:

```cpp
double price = 19.99;

int first = static_cast<int>(price); // Explicit conversion: 19
int second(price);                   // Direct initialization: 19
int third{static_cast<int>(price)};  // Brace initialization after explicit conversion: 19
```

For ordinary numeric conversion, prefer `static_cast` because it is easy to search for and shows intent.

```cpp
int completed = 3;
int total = 4;

// INVALID LOGIC: integer division happens first, so result is 0.
double wrongPercentage = completed / total * 100;

// VALID: convert one operand before division.
double percentage = static_cast<double>(completed) / total * 100; // 75.0
```

---

## Named C++ casts

| Cast | Main use | Runtime check? | Typical safety |
| --- | --- | --- | --- |
| `static_cast<T>` | Numeric conversion; safe/known inheritance conversion | No | Usually the default explicit cast |
| `dynamic_cast<T>` | Downcasting polymorphic base pointers/references | Yes | Safe runtime-checked inheritance cast |
| `const_cast<T>` | Add/remove `const` or `volatile` | No | Rare; dangerous if you modify an originally `const` object |
| `reinterpret_cast<T>` | Low-level bit/address reinterpretation | No | Very low-level; avoid in normal application code |

### 3. `static_cast<T>`

Use `static_cast` for conversions C++ can verify at compile time.

#### Numeric conversion

```cpp
double ratio = 2.75;
int roundedDown = static_cast<int>(ratio); // 2; decimal part is discarded.

int count = 500;
double exactEnough = static_cast<double>(count); // 500.0
```

#### Safe inheritance conversion: derived → base (upcast)

```cpp
class Animal {
public:
    virtual ~Animal() = default;
};

class Dog : public Animal {};

Dog dog;

// Always safe: every Dog is also an Animal.
Animal* animal = static_cast<Animal*>(&dog);
```

```text
Dog object
┌─────────────────────────┐
│ Animal part             │  ◄── Animal* can view this safely
│ Dog-specific part       │
└─────────────────────────┘
```

#### Risky inheritance conversion: base → derived (downcast)

```cpp
Dog realDog;
Animal* animal = &realDog;

// Compiles, but static_cast does NOT check the real object at runtime.
Dog* dog = static_cast<Dog*>(animal);
```

This is only safe when you already know `animal` truly points to a `Dog`.

```cpp
Animal plainAnimal;
Animal* animal = &plainAnimal;

// Compiles, but is dangerous: the real object is NOT a Dog.
Dog* dog = static_cast<Dog*>(animal);

// Using dog as a Dog is undefined behavior.
```

For an uncertain downcast, use `dynamic_cast` instead.

```text
             inheritance (“is-a”)
Dog ─────────────────────────────▷ Animal ◁──────────────────────────── Cat
                                      │
                                      │ Animal* / Animal& can refer to
                                      │ either derived object
                                      ▼
                           dynamic_cast<Dog*>
                           succeeds for Dog;
                           returns nullptr for Cat
```

> Rule: **Use `static_cast` for an upcast. Do not use it to “guess” a derived type.**

---

### 4. `dynamic_cast<T>`: checked downcasting

`dynamic_cast` asks at runtime: “Is this object really the requested derived type?”

The base class must be **polymorphic**—it needs at least one virtual function. A virtual destructor is enough.

```cpp
#include <iostream>

class Animal {
public:
    virtual ~Animal() = default; // Makes Animal polymorphic.
};

class Dog : public Animal {
public:
    void bark() { std::cout << "Woof\n"; }
};

class Cat : public Animal {};

void makeDogBark(Animal* animal) {
    // Pointer dynamic_cast returns nullptr if the object is not a Dog.
    if (Dog* dog = dynamic_cast<Dog*>(animal)) {
        dog->bark(); // Valid: C++ confirmed that it is a Dog.
    } else {
        std::cout << "This animal is not a Dog\n";
    }
}

int main() {
    Dog dog;
    Cat cat;

    makeDogBark(&dog); // Woof
    makeDogBark(&cat); // This animal is not a Dog
}
```

```text
Animal* ──► Dog object  → dynamic_cast<Dog*> returns a valid Dog*
Animal* ──► Cat object  → dynamic_cast<Dog*> returns nullptr
```

For references, failure throws `std::bad_cast` instead of returning `nullptr`:

```cpp
#include <typeinfo>

Animal& animal = cat;

try {
    Dog& dog = dynamic_cast<Dog&>(animal); // Throws: cat is not a Dog.
    dog.bark();
} catch (const std::bad_cast&) {
    std::cout << "Not a Dog\n";
}
```

Prefer the pointer form when failure is an expected possibility—it is easy to test for `nullptr`.

---

### 5. `const_cast<T>`: add or remove constness

`const_cast` changes only whether the compiler sees an object as `const`; it does not change the object's real nature.

```cpp
void legacyPrint(char* text); // Imagine an old API that promises not to modify text.

void printMessage(const char* message) {
    // Sometimes used to call a legacy API. Only safe if legacyPrint does not write.
    legacyPrint(const_cast<char*>(message));
}
```

The dangerous mistake:

```cpp
const int original = 10;

int* writable = const_cast<int*>(&original);

// INVALID BEHAVIOR: original was truly declared const.
// *writable = 20; // Undefined behavior.
```

This can be valid only when the original object was not `const`:

```cpp
int original = 10;                 // Not truly const.
const int* readOnly = &original;   // View it as const.

int* writable = const_cast<int*>(readOnly);
*writable = 20;                    // Valid: original itself was mutable.
```

> Rule: **Use `const_cast` only for a well-understood legacy/API boundary. Never use it to modify an object declared `const`.**

---

### 6. `reinterpret_cast<T>`: low-level representation conversion

`reinterpret_cast` can reinterpret an address or bits as an unrelated type. It is for low-level systems code, not ordinary application code.

```cpp
#include <cstdint>

int value = 42;

// Converts a pointer address to an integer-sized value.
std::uintptr_t address = reinterpret_cast<std::uintptr_t>(&value);

// Converts it back to the original pointer type.
int* sameValue = reinterpret_cast<int*>(address);
```

This does **not** safely convert an `int` into a different, unrelated object type.

```cpp
int number = 42;

// Avoid: treating int memory as a double object is undefined behavior.
// double* wrong = reinterpret_cast<double*>(&number);
// std::cout << *wrong;
```

If you need to copy raw bits between same-sized trivially copyable types in C++20, prefer `std::bit_cast` from `<bit>` rather than pointer tricks.

---

### 7. C-style casts: avoid them

```cpp
double price = 19.99;

// Avoid: it could behave like static_cast, const_cast, or reinterpret_cast.
int value = (int)price;

// Prefer: visible and searchable intent.
int betterValue = static_cast<int>(price);
```

Named casts help reviewers understand risk:

```text
static_cast       → ordinary conversion
dynamic_cast      → runtime-checked inheritance conversion
const_cast        → constness changed: inspect carefully
reinterpret_cast  → low-level conversion: inspect very carefully
```

## Quick decision guide

| You need to… | Prefer |
| --- | --- |
| Convert between numeric types | `static_cast<T>(value)` |
| Convert derived pointer/reference to base | Usually implicit conversion or `static_cast` |
| Convert base pointer/reference to derived, with uncertainty | `dynamic_cast<T*>` / `dynamic_cast<T&>` |
| Call a legacy API that wrongly lacks `const` | Rare, carefully justified `const_cast` |
| Work with raw addresses/bits in low-level code | Rare `reinterpret_cast`; consider `std::bit_cast` |
| Make code short using `(T)value` | Do not; use a named cast instead |

> Memory rule: **Default to `static_cast` for ordinary explicit conversion. Use `dynamic_cast` when you need C++ to verify the real derived type at runtime. Treat `const_cast` and `reinterpret_cast` as warning signs.**
