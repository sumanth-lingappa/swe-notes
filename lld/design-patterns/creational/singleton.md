---
layout: default
title: Singleton
---

# Singleton: LLD quick reference

## What is a singleton?

A singleton is a class/type designed to have **one shared object** for the lifetime of the program.

Typical examples: application configuration, a logger, or a metrics registry.

The two goals are:

1. Stop other code from creating extra objects.
2. Provide one controlled way to get the shared object.

> Core idea: **A singleton gives controlled global access to exactly one instance.**

## Class and object relationship

```text
OrderService ──calls──┐
EmailWorker  ──calls───┼──► AppConfig::instance()
WebHandler   ──calls───┘             │
                                     │ returns a reference to
                                     ▼
                              one AppConfig object

The constructor is private, so clients cannot create another AppConfig.
The arrows all lead to the same object, not one object per caller.
```

---

## C++: function-local static (Meyers singleton)

```cpp
#include <iostream>
#include <string>

class AppConfig {
public:
    // The only normal way to access the one object.
    static AppConfig& instance() {
        // Created once, on the first call to instance().
        // Since C++11, this initialization is thread-safe.
        static AppConfig config;
        return config;
    }

    const std::string& getEnvironment() const {
        return environment;
    }

    // Copy constructor: forbids making a second object by copying.
    // Forbidden: AppConfig copy = AppConfig::instance();
    AppConfig(const AppConfig&) = delete;

    // Copy assignment: forbids copying data into another AppConfig object.
    // Forbidden: existingConfig = AppConfig::instance();
    AppConfig& operator=(const AppConfig&) = delete;

    // These two are optional here: deleting copy operations normally prevents
    // automatic move generation. They make the intent explicit.
    // Forbidden: AppConfig moved = std::move(AppConfig::instance());
    AppConfig(AppConfig&&) = delete;

    // Forbidden: existingConfig = std::move(AppConfig::instance());
    AppConfig& operator=(AppConfig&&) = delete;

private:
    // Private: outside code cannot write `AppConfig config;`.
    AppConfig() = default;

    // No custom destructor needed: std::string cleans up its own memory.
    std::string environment = "production";
};

int main() {
    AppConfig& first = AppConfig::instance();
    AppConfig& second = AppConfig::instance();

    std::cout << first.getEnvironment() << '\n'; // production
    std::cout << (&first == &second) << '\n';    // 1 (same object)
}
```

### C++ Key points

- `static AppConfig config` is a **function-local static**: there is one object, constructed the first time execution reaches that line.
- Its initialization is thread-safe since **C++11**.
- The object is automatically destroyed during program shutdown.
- A `private` constructor blocks ordinary construction outside the class.
- Deleted copy operations are the important protection against accidental duplicate objects.
- This design does not need `new`, `delete`, or a custom destructor.

### How C++ application code uses it

**Singleton access** — convenient, but `OrderService` now depends on the global `AppConfig`:

```cpp
// In OrderService
std::string env = AppConfig::instance().getEnvironment();

// In EmailWorker
log(AppConfig::instance().getEnvironment());
```

**Dependency injection** — an alternative for code that is easier to test. This is an ordinary config class, not the singleton above:

```cpp
class Config {
public:
    explicit Config(std::string environment)
        : environment(environment) {}

    const std::string& getEnvironment() const { return environment; }

private:
    std::string environment;
};

// Created once in main/composition root, then passed by reference.
Config config("production");
OrderService orderService(config);
EmailWorker emailWorker(config);
```

Do **not** write `AppConfig config("production");` for the singleton: its constructor is private on purpose.

Compile:

```bash
g++ -std=c++17 singleton.cpp -o singleton
./singleton
```

---

## Python: cached instance

In Python, a module is often the simplest singleton. This class version is useful when you want a class-based singleton.

```python
import copy
import threading


class AppConfig:
    # Holds the one shared object. None means it has not been created yet.
    _instance = None

    # Makes first-time creation safe if multiple threads call AppConfig().
    _lock = threading.Lock()

    def __new__(cls):
        # __new__ creates (or returns) an object before __init__ runs.
        if cls._instance is None:
            with cls._lock:
                # Check again after acquiring the lock.
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance.environment = "production"

        # Every AppConfig() call returns the same object.
        return cls._instance

    def get_environment(self):
        return self.environment

    # Python has no C++-style `= delete` for copying.
    # Copying returns the same singleton object instead.
    def __copy__(self):
        return self

    def __deepcopy__(self, memo):
        return self


first = AppConfig()
second = AppConfig()

print(first.get_environment())  # production
print(first is second)          # True (same object)

third = copy.copy(first)
print(first is third)           # True
```

### Python Key points

- `__new__` returns the object; it is the place to enforce one cached instance.
- `is` compares identity: `first is second` checks that both names refer to the same object.
- Python has no truly private constructor like C++; this pattern is partly enforced by convention.
- In production Python, a module-level object is usually simpler:

```python
# app_config.py
environment = "production"
```

### How Python application code uses it

**Singleton access:**

```python
# in order_service.py
env = AppConfig().get_environment()

# in email_worker.py
log(AppConfig().get_environment())
```

**Dependency injection** — use an ordinary configuration class, not the singleton class above:

```python
class Config:
    def __init__(self, environment):
        self.environment = environment

    def get_environment(self):
        return self.environment


# Created once at application startup, then passed in.
config = Config("production")
order_service = OrderService(config)
email_worker = EmailWorker(config)
```

---

## Go: `sync.Once`

```go
package main

import (
	"fmt"
	"sync"
)

type AppConfig struct {
	// Lowercase: private outside this Go package.
	environment string
}

func (config *AppConfig) GetEnvironment() string {
	return config.environment
}

var (
	// Pointer to the one shared AppConfig object.
	appConfig *AppConfig

	// Guarantees that its Do function runs once, safely across goroutines.
	appConfigOnce sync.Once
)

// The normal access point for the singleton.
func GetAppConfig() *AppConfig {
	appConfigOnce.Do(func() {
		// This initialization function runs only once.
		appConfig = &AppConfig{
			environment: "production",
		}
	})

	return appConfig
}

func main() {
	first := GetAppConfig()
	second := GetAppConfig()

	fmt.Println(first.GetEnvironment()) // production
	fmt.Println(first == second)        // true (same pointer)
}
```

### Go Key points

- `sync.Once` runs its initialization function exactly once, even with multiple goroutines.
- `GetAppConfig()` returns the same `*AppConfig` pointer each time.
- Lowercase names are unexported outside the package, so `environment` cannot be changed by other packages directly.
- Go commonly uses package-level variables and functions rather than a class with a private constructor.

### How Go application code uses it

**Singleton access:**

```go
// In OrderService
env := GetAppConfig().GetEnvironment()

// In EmailWorker
log(GetAppConfig().GetEnvironment())
```

**Dependency injection** — create an ordinary config once and pass its pointer:

```go
type Config struct {
	Environment string
}

func NewConfig(environment string) *Config {
	return &Config{Environment: environment}
}

// Created once in main/composition root, then passed in.
config := NewConfig("production")
orderService := NewOrderService(config)
emailWorker := NewEmailWorker(config)
```

Run:

```bash
go run singleton.go
```

---

## Same idea, three languages

| Language | One-instance mechanism | Normal access |
| --- | --- | --- |
| C++ | Function-local `static` | `AppConfig::instance()` |
| Python | Cached class variable | `AppConfig()` |
| Go | `sync.Once` + package variable | `GetAppConfig()` |

---

## When to use—or avoid—a singleton

Use it when there really should be one shared service or configuration source.

Be cautious: a singleton is global state. It can make tests harder because code silently depends on a shared object. For services that may need different implementations in tests (for example, file storage or databases), dependency injection is often easier to test.

> Concise summary: “I use a singleton when one shared instance is genuinely required. For replaceable dependencies, I prefer passing an interface/dependency explicitly.”

## Other common singleton examples

| Example | Why one shared instance can make sense |
| --- | --- |
| Logger | Centralizes log formatting and output destination. |
| Metrics registry | Keeps one consistent registry of counters and gauges. |
| Application configuration | Provides a single read-mostly configuration source. |
| Thread pool | Coordinates a limited, shared set of worker threads. |

Use these carefully: many can also be passed explicitly as dependencies, which makes tests easier.
