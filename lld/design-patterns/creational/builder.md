---
layout: default
title: Builder
---

# Builder pattern: LLD quick reference

## What problem does Builder solve?

Use Builder when an object has a few **required** fields but many **optional** fields.

Without it, constructors become hard to read:

```cpp
// What do 30, "Bengaluru", and true mean without checking the class?
UserProfile("Asha", "asha@example.com", 30, "Bengaluru", "C++ learner", true);
```

Adding overloaded constructors creates a **telescoping constructor** problem: many constructors for different combinations of optional arguments.

Builder replaces that with named, step-by-step construction:

```text
required values → Builder → optional named methods → build() → finished object
```

```cpp
UserProfile profile = UserProfile::Builder("Asha", "asha@example.com")
    .withAge(30)
    .withLocation("Bengaluru")
    .enableNotifications()
    .build();
```

> Core idea: **Builder constructs a complex object step by step, keeping required and optional values clear and readable.**

---

## C++: fluent nested builder

```cpp
#include <iostream>
#include <stdexcept>
#include <string>
#include <utility>

class UserProfile {
public:
    // Builder is declared inside UserProfile because it builds UserProfile.
    class Builder {
    public:
        // Required fields are supplied when the builder is created.
        Builder(std::string name, std::string email)
            : name_(std::move(name)), email_(std::move(email)) {}

        // Every optional-setting method returns the SAME builder.
        // Returning Builder& enables method chaining.
        Builder& withAge(int age) {
            age_ = age;
            return *this;
        }

        Builder& withLocation(std::string location) {
            location_ = std::move(location);
            return *this;
        }

        Builder& withBio(std::string bio) {
            bio_ = std::move(bio);
            return *this;
        }

        Builder& enableNotifications() {
            notificationsEnabled_ = true;
            return *this;
        }

        // build() validates inputs and creates the finished UserProfile.
        UserProfile build() const {
            if (name_.empty()) {
                throw std::invalid_argument("name is required");
            }

            if (email_.find('@') == std::string::npos) {
                throw std::invalid_argument("email must contain @");
            }

            // The private constructor prevents direct creation by callers.
            return UserProfile(name_, email_, age_, location_, bio_, notificationsEnabled_);
        }

    private:
        // Builder stores values while the object is being assembled.
        std::string name_;
        std::string email_;
        int age_ = 0;
        std::string location_;
        std::string bio_;
        bool notificationsEnabled_ = false;
    };

    // Factory-like entry point: makes required fields obvious.
    static Builder builder(std::string name, std::string email) {
        return Builder(std::move(name), std::move(email));
    }

    void print() const {
        std::cout << "name=" << name_
                  << ", email=" << email_
                  << ", age=" << age_
                  << ", location=" << location_
                  << ", notifications=" << notificationsEnabled_
                  << '\n';
    }

private:
    // Private: clients must use the Builder to create a complete object.
    UserProfile(std::string name, std::string email, int age,
                std::string location, std::string bio, bool notificationsEnabled)
        : name_(std::move(name)),
          email_(std::move(email)),
          age_(age),
          location_(std::move(location)),
          bio_(std::move(bio)),
          notificationsEnabled_(notificationsEnabled) {}

    // Finished object's state. No public setters: it is immutable after build().
    std::string name_;
    std::string email_;
    int age_;
    std::string location_;
    std::string bio_;
    bool notificationsEnabled_;
};

int main() {
    // Required values first; optional values only when needed.
    UserProfile profile = UserProfile::builder("Asha", "asha@example.com")
        .withAge(30)
        .withLocation("Bengaluru")
        .enableNotifications()
        .build();

    profile.print();
}
```

### How to read the C++ chain

```text
UserProfile::builder(...)  → makes a temporary Builder
.withAge(...)              → updates it; returns Builder& (the same builder)
.withLocation(...)         → updates it; returns Builder&
.build()                   → validates and returns UserProfile
```

### C++ Key points

- The product is `UserProfile`; the builder temporarily holds values until `build()`.
- Named methods make optional values self-documenting.
- `build()` is the right place to validate required fields and cross-field rules.
- A private product constructor encourages callers to use the builder.
- Returning `Builder&` enables fluent chaining and avoids copying the builder at each step.
- `std::move` transfers strings into their final storage when appropriate; callers do not need to understand it to use Builder.

Compile:

```bash
g++ -std=c++17 builder.cpp -o builder
./builder
```

### How C++ application code uses it

```cpp
// In a registration service: only set optional fields that were supplied.
UserProfile profile = UserProfile::builder(request.name, request.email)
    .withLocation(request.location)
    .enableNotifications()
    .build();

userRepository.save(profile);
```

---

## Python: fluent builder

```python
class UserProfile:
    # The final product. Its constructor is treated as internal by convention.
    def __init__(self, name, email, age=0, location="", bio="", notifications_enabled=False):
        self.name = name
        self.email = email
        self.age = age
        self.location = location
        self.bio = bio
        self.notifications_enabled = notifications_enabled

    def __repr__(self):
        return f"UserProfile(name={self.name!r}, email={self.email!r})"


class UserProfileBuilder:
    # Required fields are required when creating the builder.
    def __init__(self, name, email):
        self._name = name
        self._email = email

        # Defaults for optional fields.
        self._age = 0
        self._location = ""
        self._bio = ""
        self._notifications_enabled = False

    # Each method updates the builder and returns self for chaining.
    def with_age(self, age):
        self._age = age
        return self

    def with_location(self, location):
        self._location = location
        return self

    def with_bio(self, bio):
        self._bio = bio
        return self

    def enable_notifications(self):
        self._notifications_enabled = True
        return self

    def build(self):
        # Validate before creating the final object.
        if not self._name:
            raise ValueError("name is required")
        if "@" not in self._email:
            raise ValueError("email must contain @")

        return UserProfile(
            self._name,
            self._email,
            self._age,
            self._location,
            self._bio,
            self._notifications_enabled,
        )


profile = (
    UserProfileBuilder("Asha", "asha@example.com")
    .with_age(30)
    .with_location("Bengaluru")
    .enable_notifications()
    .build()
)

print(profile)  # UserProfile(name='Asha', email='asha@example.com')
```

### Python note

Python often uses keyword arguments or `dataclasses` for this problem:

```python
# Readable without a Builder when there are only a few optional values.
profile = UserProfile(name="Asha", email="asha@example.com", location="Bengaluru")
```

Use a builder when object creation needs validation, many construction steps, or a fluent domain-specific API—not merely because the pattern exists.

### How Python application code uses it

```python
# In registration_service.py
profile = (
    UserProfileBuilder(request.name, request.email)
    .with_location(request.location)
    .enable_notifications()
    .build()
)

user_repository.save(profile)
```

---

## Go: builder methods returning the builder

```go
package main

import (
	"errors"
	"fmt"
)

// Final product.
type UserProfile struct {
	Name                 string
	Email                string
	Age                  int
	Location             string
	Bio                  string
	NotificationsEnabled bool
}

// Builder holds state while the product is being assembled.
type UserProfileBuilder struct {
	name                 string
	email                string
	age                  int
	location             string
	bio                  string
	notificationsEnabled bool
}

// Required fields are provided at builder creation.
func NewUserProfileBuilder(name string, email string) *UserProfileBuilder {
	return &UserProfileBuilder{name: name, email: email}
}

// Each method changes the builder and returns it, enabling chaining.
func (builder *UserProfileBuilder) WithAge(age int) *UserProfileBuilder {
	builder.age = age
	return builder
}

func (builder *UserProfileBuilder) WithLocation(location string) *UserProfileBuilder {
	builder.location = location
	return builder
}

func (builder *UserProfileBuilder) EnableNotifications() *UserProfileBuilder {
	builder.notificationsEnabled = true
	return builder
}

// Build validates and returns the complete object.
func (builder *UserProfileBuilder) Build() (*UserProfile, error) {
	if builder.name == "" {
		return nil, errors.New("name is required")
	}

	if !containsAtSign(builder.email) {
		return nil, errors.New("email must contain @")
	}

	return &UserProfile{
		Name:                 builder.name,
		Email:                builder.email,
		Age:                  builder.age,
		Location:             builder.location,
		Bio:                  builder.bio,
		NotificationsEnabled: builder.notificationsEnabled,
	}, nil
}

func containsAtSign(text string) bool {
	for _, character := range text {
		if character == '@' {
			return true
		}
	}
	return false
}

func main() {
	profile, err := NewUserProfileBuilder("Asha", "asha@example.com").
		WithAge(30).
		WithLocation("Bengaluru").
		EnableNotifications().
		Build()

	if err != nil {
		panic(err)
	}

	fmt.Println(profile.Name) // Asha
}
```

### Go Key points

- Builder methods use pointer receivers because they update one builder object.
- They return `*UserProfileBuilder` to support chaining.
- `Build()` returns `(*UserProfile, error)` because Go commonly returns validation errors instead of throwing exceptions.
- For a small struct, a struct literal with named fields is often clearer than Builder:

```go
profile := UserProfile{Name: "Asha", Email: "asha@example.com", Location: "Bengaluru"}
```

### How Go application code uses it

```go
// In RegistrationService
profile, err := NewUserProfileBuilder(request.Name, request.Email).
	WithLocation(request.Location).
	EnableNotifications().
	Build()

if err != nil {
	return err
}

return userRepository.Save(profile)
```

---

## Builder versus similar ideas

| Pattern / technique | Main purpose |
| --- | --- |
| Builder | Construct one complex object step by step, with named optional choices. |
| Factory | Choose which kind of object to create. |
| Constructor | Quickly create an object when required fields are few and obvious. |
| Named parameters / keyword arguments | Often enough when the language supports them and validation is simple. |

## When to use—or avoid—Builder

Use Builder when:

- There are many optional fields.
- Constructor arguments would be confusing, especially when several share a type such as `std::string`.
- Construction needs validation or steps in a clear order.
- You want the finished object to be immutable after `build()`.

Avoid Builder when:

- The object has only a few clear arguments.
- A struct/object literal or keyword arguments already make construction obvious.
- The Builder would add more boilerplate than value.

> Concise summary: “I use Builder to avoid telescoping constructors and make complex construction readable. I validate in `build()`, then return a complete object.”

## Other common Builder examples

| Product | Required inputs | Typical optional inputs |
| --- | --- | --- |
| HTTP request | URL | Headers, query parameters, body, timeout, retry policy. |
| Database connection | Host/database | Port, credentials, TLS, connection-pool size. |
| Pizza order | Size | Toppings, crust, extra cheese, delivery instructions. |
| Test fixture | Main subject | Fake dependencies, seed data, feature flags. |
